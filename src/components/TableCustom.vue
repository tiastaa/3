<template>
  <div>
    <div class="table">
      <div v-for="(_, id) in arr" :key="id" class="cell">
        <input
          type="number"
          name="number"
          :id="id"
          v-model="arr1[id]"
          class="width2"
        />
      </div>
    </div>
    <div class="table">
      <div v-for="(_, id) in arrY" :key="id" class="cell">
        <input
          type="number"
          name="number"
          :id="id"
          v-model="arr2[id]"
          class="width2"
        />
      </div>
    </div>
    Elements = {{ size }}
    <div>Residual Term Lagrange = {{ maxL }}</div>
    <div>Residual Term Newton = {{ maxN }}</div>
    <button @click="button1">Lagrange</button>
    <button @click="button2">Newton</button>
    <button @click="button3">Newton Forward</button>
    <button @click="button4">Newton Back</button>
    <button @click="button5">Newton All</button>
  </div>
</template>

<script>
export default {
  name: "TableCustom",
  data() {
    return {
      size: 4,
      arr1: [],
      arr2: [],
      x: 0,
      sum: 0,
      n: 10,
      a: 3,
      b: 4,
      m: 10000,
      maxL: 0,
      maxN: 0,
    };
  },
  methods: {
    createMatrix(arr) {
      this.arr1 = arr;
    },
    createMatrix2(arr) {
      this.arr2 = arr;
    },
    Lagrange(x) {
      let sum = 0;
      let arrX = this.arr1;
      let arrY = this.arr2;
      for (let i = 0; i < this.size; i++) {
        let arrX2 = [...arrX];
        arrX2.splice(i, 1);
        let a = 1;
        let b = 1;
        for (let j = 0; j < this.size - 1; j++) {
          a *= x - arrX2[j];
          b *= arrX[i] - arrX2[j];
        }
        sum += arrY[i] * (a / b);
      }
      return sum;
    },
    myRandomInts(quantity, min, max) {
      let arr = [];
      while (arr.length < quantity) {
        var candidateInt = Math.random() * (max - min) + min;
        if (arr.indexOf(candidateInt) === -1) arr.push(candidateInt);
      }
      return arr;
    },
    Newton(x) {
      let arrX = this.arr1;
      let arrY = this.arr2;
      let arrRes = [[...arrY]];
      for (let i = 0; i < this.n - 1; i++) {
        let arrT = [];
        for (let j = 0; j < this.n - i - 1; j++) {
          let el =
            (arrRes[i][j + 1] - arrRes[i][j]) / (arrX[j + 1 + i] - arrX[j]);
          arrT.push(el);
        }
        arrRes.push(arrT);
      }

      let N = 0;
      for (let i = 0; i < arrRes.length; i++) {
        let m = arrRes[i][0];
        for (let j = 0; j < i; j++) {
          m *= x - arrX[j];
        }
        N += m;
      }
      return N;
    },
    NewtonF(x) {
      let arrX = this.arr1;
      let arrY = this.arr2;
      let arrRes = [[...arrY]];
      for (let i = 0; i < this.n - 1; i++) {
        let arrT = [];
        for (let j = 0; j < this.n - i - 1; j++) {
          let el = arrRes[i][j + 1] - arrRes[i][j];
          arrT.push(el);
        }
        arrRes.push(arrT);
      }

      let N = 0;
      for (let i = 0; i < arrRes.length; i++) {
        let m = arrRes[i][0];
        for (let j = 0; j < i; j++) {
          m *= x - arrX[j];
        }
        N += m / (this.factorialize(i) * this.h ** i);
      }
      return N;
    },
    NewtonB(x) {
      let arrX = this.arr1;
      let arrY = this.arr2;
      let arrRes = [[...arrY]];
      for (let i = 0; i < this.n - 1; i++) {
        let arrT = [];
        for (let j = 0; j < this.n - i - 1; j++) {
          let el = arrRes[i][j + 1] - arrRes[i][j];
          arrT.push(el);
        }
        arrRes.push(arrT);
      }

      let N = 0;
      for (let i = 0; i < arrRes.length; i++) {
        let m = arrRes[i][arrRes[i].length - 1];
        for (let j = 0; j < i; j++) {
          m *= x - arrX[arrRes.length - j - 1];
        }
        N += m / (this.factorialize(i) * this.h ** i);
      }
      return N;
    },
    Cheshibov(i) {
      let el1 = (this.b + this.a) / 2;
      let el2 =
        ((this.b - this.a) / 2) *
        Math.cos(((2 * i + 1) / (2 * (this.n + 1))) * Math.PI);
      return el1 + el2;
    },
    factorialize(num) {
      if (num < 0) return -1;
      else if (num == 0) return 1;
      else {
        return num * this.factorialize(num - 1);
      }
    },
    f(x) {
      return x ** 2 - x ** 3;
    },
    button1() {
      let arrF1 = [];
      let arrCheshibov = this.arrCheshibov;
      for (let i = 0; i < this.n + 1; i++) {
        arrF1.push(this.f(arrCheshibov[i]));
      }
      this.arr1 = [...arrCheshibov];
      this.arr2 = [...arrF1];
      this.size = this.n + 1;

      let max = 0;
      for (let i = 0; i < this.m; i++) {
        let el = this.a + ((this.b - this.a) / this.m) * i;
        let num1 = this.f(el);
        let num2 = this.Lagrange(el);
        let el2 = num1 - num2;

        if (max < Math.abs(el2)) {
          max = Math.abs(el2);
        }
      }
      this.maxL = max;
    },
    button2() {
      let arrX = this.myRandomInts(this.n + 1, this.a, this.b);
      arrX.sort((a, b) => a - b);
      let arrY = arrX.map((el) => this.f(el));
      this.arr1 = [...arrX];
      this.arr2 = [...arrY];
      this.size = this.n + 1;

      let max = 0;
      for (let i = 0; i < this.m; i++) {
        let el = this.a + ((this.b - this.a) / this.m) * i;
        let num1 = this.f(el);
        let num2 = this.Newton(el);
        let el2 = num1 - num2;

        if (max < Math.abs(el2)) {
          max = Math.abs(el2);
        }
      }
      this.maxN = max;
    },
    button3() {
      let arrX = [];
      for (let i = 0; i < this.n; i++) {
        arrX.push(this.a + i * this.h);
      }
      let arrY = arrX.map((el) => this.f(el));
      this.arr1 = [...arrX];
      this.arr2 = [...arrY];
      this.size = this.n;

      let max = 0;
      for (let i = 0; i < this.m; i++) {
        let el = this.a + ((this.b - this.a) / this.m) * i;
        let num1 = this.f(el);
        let num2 = this.NewtonF(el);
        let el2 = num1 - num2;

        if (max < Math.abs(el2)) {
          max = Math.abs(el2);
        }
      }
      this.max = max;
    },
    button4() {
      let arrX = [];
      for (let i = 0; i < this.n; i++) {
        arrX.push(this.a + i * this.h);
      }
      let arrY = arrX.map((el) => this.f(el));
      this.arr1 = [...arrX];
      this.arr2 = [...arrY];
      this.size = this.n;

      let max = 0;
      for (let i = 0; i < this.m; i++) {
        let el = this.a + ((this.b - this.a) / this.m) * i;
        let num1 = this.f(el);
        let num2 = this.NewtonB(el);
        let el2 = num1 - num2;

        if (max < Math.abs(el2)) {
          max = Math.abs(el2);
        }
      }
      this.max = max;
    },
    button5() {
      let arrX = [];
      for (let i = 0; i < this.n; i++) {
        arrX.push(this.a + i * this.h);
      }
      let arrY = arrX.map((el) => this.f(el));
      this.arr1 = [...arrX];
      this.arr2 = [...arrY];
      this.size = this.n;

      let max = 0;
      for (let i = 0; i < this.m; i++) {
        let el = this.a + ((this.b - this.a) / this.m) * i;
        let num1 = this.f(el);
        let num2 = this.Lagrange(el);
        let el2 = num1 - num2;

        if (max < Math.abs(el2)) {
          max = Math.abs(el2);
        }
      }
      this.maxL = max;
      max = 0;

      for (let i = 0; i < this.m; i++) {
        let el = this.a + ((this.b - this.a) / this.m) * i;
        let num1 = this.f(el);
        let num2 = this.Newton(el);
        let el2 = num1 - num2;

        if (max < Math.abs(el2)) {
          max = Math.abs(el2);
        }
      }
      this.maxN = max;
      max = 0;

      for (let i = 0; i < this.m; i++) {
        let el = this.a + ((this.b - this.a) / this.m) * i;
        let num1 = this.f(el);
        let num2 = this.NewtonF(el);
        let el2 = num1 - num2;

        if (max < Math.abs(el2)) {
          max = Math.abs(el2);
        }
      }
      this.maxNF = max;
      max = 0;

      for (let i = 0; i < this.m; i++) {
        let el = this.a + ((this.b - this.a) / this.m) * i;
        let num1 = this.f(el);
        let num2 = this.NewtonB(el);
        let el2 = num1 - num2;

        if (max < Math.abs(el2)) {
          max = Math.abs(el2);
        }
      }
      this.max = max;
    },
  },
  computed: {
    arr() {
      let arr = [];
      for (let i = 0; i < this.size; i++) {
        arr.push(this.arr1[i] ?? 0);
      }
      this.createMatrix(arr);
      return arr;
    },
    arrY() {
      let arr2 = [];
      for (let i = 0; i < this.size; i++) {
        arr2.push(this.arr2[i] ?? 0);
      }
      this.createMatrix2(arr2);
      return arr2;
    },
    arrCheshibov() {
      let arr = [];
      for (let i = 0; i < this.n + 1; i++) {
        arr.push(this.Cheshibov(i));
      }
      return arr;
    },
    h() {
      return (this.b - this.a) / this.n;
    },
    sumEl() {
      return this.arr1.reduce(function (a, b) {
        return Number(a) + Number(b);
      });
    },
    sumEl2() {
      return this.arr2.reduce(function (a, b) {
        return Number(a) + Number(b);
      });
    },
  },
};
</script>

<style scoped>
.table {
  display: flex;
  flex-direction: row;
  margin: 10px;
  /* flex-wrap: wrap; */
}
.cell {
  margin: 5px;
}
button {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 5px;
  width: 300px;
}
</style>
