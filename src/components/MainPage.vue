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
      <label for="height">Height (cm):</label>
      <input type="number" id="height" v-model="height"/>
    </div>
    <div class="input">
      <label for="gender">Gender:</label>
      <select id="gender" v-model="gender">
        <option value="Male">Male</option>
        <option value="Female">Female</option>
      </select>
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
      gender: 'Male',
      size: 0,
    }
  },
  created() {
    window.trained = false;
  },
  methods: {
    train: async () => {
      if (window.trained) {
        alert('Classifier already trained!');
        return;
      }
      let bayes = require('node-bayes');
      const response = await fetch('/newtrain.csv');
      const data = await response.blob();
      const file = new File([data], name, {
        type: data.type || "text/plain",
      });
      let columnArr = [];
      let dataArrForMale = [];
      let dataArrForFemale = [];
      const reader = new FileReader();
      reader.readAsText(file);
      reader.onload = () => {
        const lines = reader.result.split('\r\n');
        lines.forEach(line => {
          const [gender, weight, height, size] = line.split(',');
          if (gender === 'Gender') {
            columnArr = ['Weight', 'Height', 'Size'];
            console.log(columnArr);
          } else if (weight && gender && height) {
            if (gender === 'Male') {
              dataArrForMale.push([parseInt(weight), parseInt(height), size]);
              console.log([parseInt(weight), parseInt(height), size])
            } else {
              dataArrForFemale.push([parseInt(weight), parseInt(height), size]);
              console.log([parseInt(weight), parseInt(height), size])
            }
          }
        });

        window.maleClassifier = new bayes.NaiveBayes({
          columns: columnArr,
          data: dataArrForMale,
          verbose: true
        });
        window.maleClassifier.train();

        window.femaleClassifier = new bayes.NaiveBayes({
          columns: columnArr,
          data: dataArrForFemale,
          verbose: true
        });
        window.maleClassifier.train();
        window.femaleClassifier.train();
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
      window.gender = this.gender;
      let answer
      if (this.gender === 'Male') {
        answer = window.maleClassifier.predict([window.weight, window.height]);
      } else {
        answer = window.femaleClassifier.predict([window.weight, window.height]);
      }
      alert('The prediction for weight ' + window.weight + ', height ' + window.height + ', gender ' + window.gender + ' is ' + answer.answer);
      console.log([window.weight, window.height], answer);
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