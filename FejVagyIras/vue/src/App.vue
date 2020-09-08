<template>
  <div id="app">
    <p>
      Linkek: <a href="kiserlet.txt" download>kiserlet.txt  </a>
      <a href="e_inf_15okt_fl.pdf" target="_blank">Feladat   </a>
      <a href="http://git.jedlik.eu/karcag.tamas/ts-fej-vagy-iras-karcag.git" target="_blank">Forrás </a>
    </p>
    <txt-reader v-on:load="src = $event" title="Kérem töltse fel a forrás (kiserlet.txt) állományt!" />
    <div id="megoldas" v-show="show">
        <p>1. feladat: <br> A pénzfeldobás eredménye: {{randomValue()}}</p>
        <p>2. feladat: <br> Tippeljen! (F/I) = <input type="text" v-model="tipp" placeholder="(F/I)"/> <br></p>
        <p> A tipp {{tipp}}. A dobás eredménye {{rnd}} volt. <br> {{userTip()}}</p>
        <p>3. feladat: <br> A kisérlet {{T.length}} dobásból állt.</p>
        <p>4. feladat: <br> A kisérlet során a fej relatív gyakorisága {{relativeFrequency(true)}}% volt.</p>
        <p>5. feladat: <br> A kisérlet során {{countOfDouble(true)}} alkalommal dobtak pontosan két fejet egymás után. </p>
        <p>6. feladat: <br> A leghosszabb tisztafej sorozat {{longestSeries(true)}} tagból áll, kezdete a(z) {{startplace}}. dobás.</p>
        <p>7. feladat: <br> FFFF: {{countOf("FFFF")}}, FFFI: {{countOf("FFFI")}}</p>
        <p>{{generatedT}}</p>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import TxtReader from "./components/TxtReader.vue";
import { isNull } from "util";
import { triggerAsyncId } from "async_hooks";

@Component({
  components: {
    TxtReader
  }
})
export default class App extends Vue {
  private src: string = "";
  private show: boolean = false;
  private T: boolean[] = [];
  private tipp: string = "";
  private rnd: string = "";
  private startplace: number = 0;
  private generatedT: string[] = this.generating();

  @Watch("src", { immediate: true, deep: true })
  onForrasChanged(val: string, oldVal: string) {
    if (val != "") {
      this.Read();
    }
  }
  /** Beolvasott fájl soronkénti feldolgozása */
  private Read(): void {
    try {
      this.src.split("\n").forEach(i => {
        const row: string = i.trim();
        if (row == "F") this.T.push(true);
        else if (row == "I") this.T.push(false);
        this.T.push;
      });
      this.show = true;
    } catch (error) {
      this.show = false;
      this.T = [];
      this.src = "";
      window.alert("Hibás forrás");
    }
  }

  /** Véletlenszerüen generálunk egy "F" vagy egy "I" értéket az 'rnd' változóba */
  private random(): void {
    let x: number = Math.random() < 0.5 ? 1 : 0;
    if (x == 0) this.rnd = "I";
    else this.rnd = "F";
  }

  //1. feladat
  /** Vissza ad egy véletlenszerüen generált "F" vagy "I" értéket */
  private randomValue(): string {
    this.random();
    return this.rnd;
  }
  //2. feladat
  /** Eldönti, hogy a felhasználó helyes értéket adott-e meg és vissza ad egy választ*/
  private userTip(): string {
    this.random();
    // eslint-disable-next-line
    if (this.tipp == this.rnd) return "Ön eltalálta!";
    else return "Ön nem találta el!";
  }
  //4. feladat
  /** A megadott érték alapján vissza adja a tömben való relativ gyakoriságát */
  private relativeFrequency(char: boolean): string {
    let count: number = 0;
    this.T.forEach(i => {
      if (char == i) count++;
    });
    let x: number = (count / this.T.length) * 100;
    return x.toFixed(2);
  }
  //5. feladat
  /** A megadott értéknek kiszámolja a tömben található párjainak számát*/
  private countOfDouble(char: boolean): number {
    let count: number = 0;
    for (let i = 1; i < this.T.length - 2; i++) {
      if (i == 0) {
        if (
          this.T[i] == char &&
          this.T[i + 1] == char &&
          this.T[i + 2] != char
        ) {
          count++;
        }
      } else if (i == this.T.length - 3) {
        if (
          this.T[i] != char &&
          this.T[i + 1] == char &&
          this.T[i + 2] == char
        ) {
          count++;
        }
      } else {
        if (
          this.T[i - 1] != char &&
          this.T[i] == char &&
          this.T[i + 1] == char &&
          this.T[i + 2] != char
        ) {
          count++;
        }
      }
    }
    return count;
  }
  //6. feladat
  /** A megadott értéknek leghosszabb sorozatát keresi meg és adja vissza*/
  private longestSeries(char: boolean) {
    let longestrow: number = 0;
    let longestrowstartplace: number = 0;
    let count: number = 0;
    let startplace: number = 0;
    let previous: boolean = false;
    for (let i = 0; i < this.T.length; i++) {
      if (this.T[i] == char && previous) {
        count++;
      }
      if (this.T[i] == char && !previous) {
        count = 1;
        startplace = i + 1;
        previous = true;
      }
      if (this.T[i] != char) {
        if (longestrow < count) {
          longestrow = count;
          longestrowstartplace = startplace;
        }
        count = 0;
        previous = false;
      }
    }
    this.startplace = longestrowstartplace;
    return longestrow;
  }
  //7. feladat
  /** Generálunk egy 1000 elemü tömböt */
  private generating(): string[] {
    let M: string[] = [];
    let s: string = "";
    for (let i = 0; i < 1000; i++) {
      for (let j = 0; j < 4; j++) {
        this.random();
        s += this.rnd;
      }
      M.push(s);
      s = "";
    }
    return M;
  }

  /** Meg számoljuk hogy az adott elemből hány darab van a M tömben*/
  private countOf(s: string): number {
    let count: number = 0;
    this.generatedT.forEach(i => {
      if (i == s) count++;
    });
    return count;
  }
}
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
}
a {
  text-decoration: none;
}
</style>
