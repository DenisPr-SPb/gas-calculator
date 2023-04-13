<template>
  <div class="container">
    <h1 class="calc__title">Gas calculator</h1>
    <div class="result__wrapper __wrapper">
      <div v-if="errorMsg" class="errors">
        {{errorMsg}}
      </div>
      <div v-else>
        <div>Сумма к оплате: {{result}} руб.</div>
        <div>Объем потребленного газа: {{diff}}м<sup>3</sup></div>
        <div>Цена за 1000м<sup>3</sup>: {{sellingPrice}} руб.</div>
      </div>
    </div>
    <div class="calc__wrapper __wrapper">
      <div>
        <input type="text" v-model="previousMeasure" placeholder="* Предыдущие показания счетчика (м3)">
      </div>
      <div>
        <input type="text" v-model="currentMeasure" placeholder="* Конечные показания счетчика (м3)">
      </div>
      <div>
        <input type="text" v-model="calories" placeholder="* Калорийность газа (ккал/м3)" >
      </div>
      <div>
        <input type="text" v-model="tempMeasure" placeholder="* Температура при которой измерены показания ">
      </div>
    </div>

    <div class="button__wrapper __wrapper">
      <div class="button" @click="calculateData">Calculate</div>
    </div>

  </div>
</template>

<script>
export default {
  data(){
    return {
      currentMeasure: '',
      previousMeasure: '',
      calories: '',
      tempMeasure: '',
      standardCalories: 7900,
      standardPrice: 4800,
      sellingPrice: 0,
      diff: 0,
      result: 0,
      errorMsg: '',
      recordingData: []
    }
  },
  methods: {
    calculateData () {
      if (!this.currentMeasure || !this.previousMeasure || !this.tempMeasure || !this.calories) {
        return this.errorMsg = `Требуется заполнить все поля со *`
      }

      this.errorMsg =''
      let measureDiff
      let gasCoefficient
      let caloriesCoefficient
      const temp = parseInt(this.tempMeasure)
      const currentMeasure = parseInt(this.currentMeasure)
      const previousMeasure = parseInt(this.previousMeasure)
      const calories = parseInt(this.calories)

      if (previousMeasure === 0) {
        measureDiff = currentMeasure
      } else if (currentMeasure < previousMeasure) {
        return this.errorMsg = `Конечные показания не могут быть меньше показаний прошлого периода`
      } else {
        measureDiff = currentMeasure - previousMeasure
      }

      this.diff = measureDiff

      if (calories < 5000 || calories > 10000) {
        return this.errorMsg = `Неверно указанны данные по каллорийности, диапазон 5000 ккал/м3 - 10000 ккал/м3: недопустимое значение ${calories} ккал/м3`
      } else {
        caloriesCoefficient = calories / this.standardCalories
      }



      if (temp === 20) {
        gasCoefficient = 1
      } else if (temp >= -20 && temp <= 0) {
        gasCoefficient = 0.8
      } else if (temp > 0 && temp < 20) {
        gasCoefficient = 1.2
      } else if (temp > 20 && temp < 30) {
        gasCoefficient = 0.95
      } else if (temp >= 30 && temp <= 50) {
        gasCoefficient = 1.32
      } else {
        return this.errorMsg = `Неверно указанны данные по температуре, диапазон от -20°C до 50°C: недопустимое значение ${this.tempMeasure}°C`
      }

      this.sellingPrice = Math.round(this.standardPrice * gasCoefficient * caloriesCoefficient)

      this.result = ( ( measureDiff * this.sellingPrice ) / 1000)
      const currentData = {
        date: Date.now(),
        standardPrice: this.standardPrice,
        sellingPrice: this.sellingPrice,
        gasCoefficient,
        caloriesCoefficient,
        calories,
        temp,
        previousMeasure,
        currentMeasure,
        measureDiff

      }

      this.recordingData.push(currentData)
      this.tempMeasure = ''
      this.currentMeasure = ''
      this.previousMeasure = ''
      this.calories = ''

      console.log(this.recordingData)
    }
  }
}
</script>

<style scoped>
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}

body {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.container {
  box-shadow: 4px 4px 59px 20px rgba(9, 144, 247, 0.1);
  border-radius: 3px;
  max-width: 50%;
  min-width: 550px;
  height: 400px;
  margin: 250px auto;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 10px 0;
}

.errors {
  color: red;
  font-size: 1.3em;
  font-weight: bold;
}

.__wrapper {
  min-width: 100%;
  margin-bottom: 15px ;
}

.result__wrapper {
  display: flex;
  flex-direction: column;
}

.result__wrapper div {
  width: 80%;
  align-self: center;
  margin-bottom: 5px;
}

.calc__wrapper {
  margin: 0 auto;
  display: flex;
  flex-direction: column;
}
.calc__wrapper div {
  width: 80%;
  align-self: center;
  margin-bottom: 5px;
}

.calc__wrapper div input{
  width: 100%;
}

.button__wrapper {
}

.button {
  width: 30%;
  margin: 10px auto;
  text-align: center;
  border: 1px solid lightblue;
  background-color: lightskyblue;
  font-size: 1.5em;
  border-radius: 5px;
  padding: 5px 10px;
}

.button:hover {
  cursor: pointer;
  color: aliceblue;
  background-color: #7fadff;
}
</style>
