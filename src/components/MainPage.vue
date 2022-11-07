<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <p>
      Click the button below to start training the classifier. You can see the training data
      <a href="/train.csv" target="_blank" rel="noopener">here</a>.
    </p>
    <button @click="train" class="button">Train</button>
    <h3>Input the data for prediction:</h3>
    <div class="input">
      <label for="weight">Weight (kg):</label>
      <input type="number" id="weight" v-model="weight"/>
    </div>
    <div class="input">
      <label for="age">Age:</label>
      <input type="number" id="age" v-model="age"/>
    </div>
    <div class="input">
      <label for="height">Height (cm):</label>
      <input type="number" id="height" v-model="height"/>
    </div>
    <button @click="predict" class="button">Predict</button>
  </div>
</template>

<script>

export default {
  name: 'MainPage',
  props: {
    msg: String
  },
  data() {
    return {
      height: 0,
      weight: 0,
      age: 0,
      size: 0,
    }
  },
  created() {
    window.trained = false;
  },
  methods: {
    train: async () => {
      let bayes = require('node-bayes');
      const response = await fetch('/train.csv');
      const data = await response.blob();
      const file = new File([data], name, {
        type: data.type || "text/plain",
      });
      let columnArr = [];
      let dataArr = [];
      const reader = new FileReader();
      reader.readAsText(file);
      reader.onload = () => {
        const lines = reader.result.split('\r\n');
        lines.forEach(line => {
          const [weight, age, height, size] = line.split(',');
          if (weight === 'weight') {
            columnArr = [weight, age, height, 'bmi', 'massSquared', size];
            console.log(columnArr);
          } else if (weight && age && height && size) {
            let bmi = weight / ((height / 100) ** 2);
            let massSquared = weight ** 2;
            dataArr.push([parseFloat(weight), parseFloat(age), Math.round(parseFloat(height)), bmi, massSquared, size]);
            console.log([parseFloat(weight), parseFloat(age), Math.round(parseFloat(height)), bmi, massSquared, size])
          }
        });
        window.cls = new bayes.NaiveBayes({
          columns: columnArr,
          data: dataArr,
          verbose: true
        });
        window.cls.train();
        alert('Training complete');
        window.trained = true;
      }
    },
    predict() {
      if (!window.trained) {
        alert('Please train the classifier first');
        return;
      }
      window.weight = this.weight;
      window.height = this.height;
      window.age = this.age;
      let bmi = window.weight / ((window.height / 100) ** 2);
      let massSquared = window.weight ** 2;
      let answer = window.cls.predict([window.weight, window.age, window.height, bmi, massSquared]);
      alert('The prediction for weight ' + window.weight + ', age ' + window.age + ', height ' + window.height + ' is ' + answer.answer);
      console.log([window.weight, window.age, window.height], answer);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

button {
  background-color: #42b983;
  border: none;
  color: white;
  padding: 15px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
}
</style>
