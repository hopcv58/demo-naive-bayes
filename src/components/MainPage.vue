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

    window.minMaleWeight = 112.903;
    window.minMaleWeightKg = 45;
    window.maxMaleWeightDiff = 269.99 - 112.903;
    window.maxMaleWeightKgDiff = 90-55;

    window.minMaleHeight = 58.4069
    window.minMaleHeightCm = 147;
    window.maxMaleHeightDiff = 78.9987 - 58.4069;
    window.maxMaleHeightCmDiff = 190-147;

    window.minFemaleWeight = 64.7001;
    window.minFemaleWeightKg = 30;
    window.maxFemaleWeightDiff = 202.237 - 64.7001;
    window.maxFemaleWeightKgDiff = 80-30;

    window.minFemaleHeight = 54.2631
    window.minFemaleHeightCm = 130;
    window.maxFemaleHeightDiff = 73.3896 - 54.2631;
    window.maxFemaleHeightCmDiff = 180-130;
  },
  methods: {
    train: async () => {
      if (window.trained) {
        alert('Classifier already trained!');
        return;
      }
      let bayes = require('node-bayes');
      const response = await fetch('/weight-height.csv');
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
          const [gender, height, weight] = line.split(',');
          if (gender === '"Gender"') {
            columnArr = ['Weight', 'Height', 'Size'];
            console.log(columnArr);
          } else if (weight && gender && height) {
            if (gender === '"Male"') {
              let weightDiffWithMin = weight - window.minMaleWeight;
              let weightKg = weightDiffWithMin * window.maxMaleWeightKgDiff / window.maxMaleWeightDiff + window.minMaleWeightKg;
              let heightDiffWithMin = height - window.minMaleHeight;
              let heightCm = heightDiffWithMin * window.maxMaleHeightCmDiff / window.maxMaleHeightDiff + window.minMaleHeightCm;
              let size = '';

              weightKg = Math.round(weightKg);
              heightCm = Math.round(heightCm);

              if (weightKg < 55) {
                if (heightCm < 160) {
                  size = 'XS';
                } else {
                  size = 'S';
                }
              } else if (weightKg < 60) {
                if (heightCm < 165) {
                  size = 'S';
                } else {
                  size = 'M';
                }
              } else if (weightKg < 65) {
                if (heightCm < 170) {
                  size = 'M';
                } else {
                  size = 'L';
                }
              } else if (weightKg < 70) {
                if (heightCm <= 174) {
                  size = 'L';
                } else {
                  size = 'XL';
                }
              } else if (weightKg <= 76) {
                if (heightCm <= 176) {
                  size = 'XL';
                } else {
                  size = 'XXL';
                }
              } else if (weightKg < 80) {
                if (heightCm <= 177) {
                  size = 'XXL';
                } else {
                  size = 'XXXL';
                }
              } else {
                size = 'XXXL'
              }
              dataArrForMale.push([weightKg, heightCm, size]);
            } else {
              let weightDiffWithMin = weight - window.minFemaleWeight;
              let weightKg = weightDiffWithMin * window.maxFemaleWeightKgDiff / window.maxFemaleWeightDiff + window.minFemaleWeightKg;
              let heightDiffWithMin = height - window.minFemaleHeight;
              let heightCm = heightDiffWithMin * window.maxFemaleHeightCmDiff / window.maxFemaleHeightDiff + window.minFemaleHeightCm;
              let size = '';

              weightKg = Math.round(weightKg);
              heightCm = Math.round(heightCm);

              if (weightKg < 38) {
                if (heightCm < 145) {
                  size = 'XS';
                } else {
                  size = 'S';
                }
              } else if (weightKg < 43) {
                if (heightCm < 153) {
                  size = 'S';
                } else {
                  size = 'M';
                }
              } else if (weightKg < 46) {
                size = 'M';
              } else if (weightKg < 53) {
                size = 'L';
              } else if (weightKg <= 57) {
                size = 'XL';
              } else if (weightKg < 66) {
                size = 'XXL';
              } else {
                size = 'XXXL'
              }
              dataArrForFemale.push([weightKg, heightCm, size]);
            }
          }
        });

        let string = ''
        for (let i = 0; i <= dataArrForMale.length; i++) {
          if (dataArrForMale[i])
          string = string + 'Male,' + dataArrForMale[i].join(',') + '\n';
        }
        for (let i = 0; i <= dataArrForFemale.length; i++) {
          if (dataArrForFemale[i])
          string = string + 'Female,' + dataArrForFemale[i].join(',') + '\n';
        }
        console.log(string);

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
