<template>
  <div class="hello">
    <h1>{{ languageSet[language].header }}</h1>
    <div class="input">
      <label for="language">{{ languageSet[language].choose }}</label>
      <select v-model="language" id="language">
        <option v-for="(value, key) in languageSet" :value="key" :key="key">{{ value.name }}</option>
      </select>
    </div>
    <h3>{{ languageSet[language]['choose-size-yourself'] }}</h3>
    <img alt="Size table" src="../assets/table.png">
    <h3>{{ languageSet[language]['input-for-prediction'] }}</h3>
    <div class="input">
      <label for="weight">{{ languageSet[language].weight }}</label>
      <input type="number" id="weight" v-model="weight"/>
    </div>
    <div class="input">
      <label for="height">{{ languageSet[language].height }}</label>
      <input type="number" id="height" v-model="height"/>
    </div>
    <div class="input">
      <label for="gender">{{ languageSet[language].gender }}</label>
      <select id="gender" v-model="gender">
        <option value="Male">{{ languageSet[language].male }}</option>
        <option value="Female">{{ languageSet[language].female }}</option>
      </select>
    </div>
    <button @click="predict" class="button">{{ languageSet[language].predict }}</button>
  </div>
</template>

<script>

export default {
  name: 'MainPage',
  data() {
    return {
      height: 0,
      weight: 0,
      gender: 'Male',
      size: 0,
      language: 'vi',
      languageSet: {
        'en' : {
          'name' : 'English',
          'choose' : 'Choose language',
          'header' : 'Clothes size predictor App',
          'choose-size-yourself' : 'You can choose size yourself by using this table',
          'input-for-prediction' : 'Or input your information and we will suggest you the best size',
          'weight' : 'Weight (kg):',
          'height' : 'Height (cm):',
          'gender' : 'Gender:',
          'male' : 'Male',
          'female' : 'Female',
          'predict' : 'Predict',
          'training' : 'Please wait when we are initializing the model...',
          'answer' : 'Our prediction for your clothes size is: '
        },
        'vi' : {
          'name' : 'Tiếng Việt',
          'choose' : 'Chọn ngôn ngữ',
          'header' : 'Ứng dụng Dự đoán kích cỡ quần áo',
          'choose-size-yourself' : 'Bạn có thể chọn kích cỡ quần áo của mình bằng bảng sau',
          'input-for-prediction' : 'Hoặc nhập thông tin của bạn và chúng tôi sẽ đưa ra kết quả dự đoán.',
          'weight' : 'Cân nặng (kg):',
          'height' : 'Chiều cao (cm):',
          'gender' : 'Giới tính:',
          'male' : 'Nam',
          'female' : 'Nữ',
          'predict' : 'Dự đoán',
          'training' : 'Vui lòng đợi trong khi chúng tôi khởi tạo mô hình...',
          'answer' : 'Kích cỡ quần áo của bạn là: '
        }
      }
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
        alert(this.languageSet[this.language].training);
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
      alert(this.languageSet[this.language].answer + answer.answer);
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
  width: 200px;
  text-align: left;
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
