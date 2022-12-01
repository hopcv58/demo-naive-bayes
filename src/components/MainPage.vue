<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
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
  async created() {
    window.trained = false;
    await this.train();
    window.trained = true;
  },
  methods: {
    train: async () => {
      if (window.trained) {
        return;
      }
      let bayes = require('node-bayes');
      const response = await fetch('/vn_train.csv');
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
        window.trained = true;
      }
    },
    predict() {
      if (!window.trained) {
        alert('Please wait when we are initializing the model');
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

.input {
  margin: 20px 0;
}
.input label {
  display: inline-block;
  width: 100px;
}
.input input {
  width: 200px;
  padding: 8px;
}
.input select {
  width: 220px;
  padding: 8px;
}
</style>
