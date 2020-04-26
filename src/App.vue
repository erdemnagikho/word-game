<template>
  <div class="container">
    <div class="card mt-4" v-if="completed">
      <div class="card-body">Yarışma bitti, puanınız {{score}}</div>
    </div>
    <div class="card mt-4" v-if="!currentQuestion">
      <div class="card-header">
        <h5 class="mb-0">Kelime Oyunu</h5>
      </div>
      <div class="card-body">Başlamak için butona bas</div>
      <div class="card-footer">
        <button class="btn btn-primary" @click="start">Yarışmaya başla</button>
      </div>
    </div>
    <div class="card mt-4" v-else>
      <div class="card-header">
        <h3 class="mb-0">{{currentQuestion.question}}</h3>
      </div>

      <div class="card-body">
        <div class="d-flex">
          <Letter
            :value="letter.letter"
            :opened="letter.opened"
            v-for="(letter, index) in letters"
            :key="index"
          />
        </div>
      </div>

      <div class="card-footer">
        <div class="d-flex">
          <div class="mr-4">Toplam Puan: {{score}}</div>
          <div class="mr-4">Soru Puan: {{letterScore}}</div>
          <div>
            Kalan Süre:
            <kbd>{{remainingTime}}</kbd> saniye
          </div>
        </div>
      </div>
      <div class="card-footer" :class="messageClass" v-if="message">{{message}}</div>
      <div class="card-footer">
        <div class="input-group">
          <input
            class="form-control"
            type="text"
            placeholder="Cevabınız"
            v-model="competitorAnswer"
            @keyup="competitorAnswer = competitorAnswer.replace(/ /, '').toLocaleUpperCase('tr')"
          />
          <div class="input-group-append">
            <button :disabled="disabledAnswerBtn || !competitorAnswer" class="btn btn-success" @click="answer">Cevap Ver</button>
          </div>
        </div>
      </div>
      <div class="card-footer">
        <button
          :disabled="disabledGiveLetterBtn"
          class="btn btn-warning"
          @click="giveLetter"
        >Harf Ver</button>
      </div>
    </div>
  </div>
</template>

<script>
import Letter from "./components/Letter";
export default {
  name: "App",
  components: { Letter },
  data() {
    return {
      questions: [
        {
          question:
            "Takım çalışmalarında ekip elemanlarının sorumluluk ve yükümlülükleri paylaşması",
          answer: "İŞBÖLÜMÜ",
          asked: false
        },
        {
          question:
            "Futbolda hem orta yuvarlak hem başlama vuruşu anlamlarına gelen söz",
          answer: "SANTRA",
          asked: false
        },
        {
          question:
            "Hastane, ev, fabrika gibi yerlerden kullanılmış ve kullanıcının artık işine yaramayan maddelerin tümü",
          answer: "ATIK",
          asked: false
        },
        {
          question:
            "Birden ortaya çıkan, aşırı ve normalin dışına taşmış korku hali",
          answer: "PANİK",
          asked: false
        },
        {
          question:
            "Genellikle ''viyol'' denen özel karton kaplarda satılan, kendinden ambalajlı bir besin",
          answer: "YUMURTA",
          asked: false
        }
      ],
      currentQuestion: null,
      letters: [],
      score: 0,
      letterScore: 0,
      competitorAnswer: "",
      completed: false,
      time: null,
      message: null,
      messageClass: "",
      messageTime: null,
      remainingTime: 0,
      disabledGiveLetterBtn: false,
      disabledAnswerBtn: false
    };
  },
  methods: {
    start() {
      this.completed = false;
      this.currentQuestion = null;
      this.score = 0;
      this.questions.map(q => {
        q.asked = false;
      });
      this.remainingTime = 240;
      this.time = setInterval(() => {
        this.remainingTime--;
        if (this.remainingTime === 0) {
          this.finish();
        }
      }, 1000);
      this.giveQuestion();
      this.showMessage("Yarışma başladı");
    },
    giveQuestion() {
      this.competitorAnswer = null;
      this.currentQuestion = this.questions.find(q => !q.asked);
      if (!this.currentQuestion) {
        this.finish();
        return;
      }
      this.letters = [];
      this.currentQuestion.answer.split("").map(l => {
        this.letters.push({
          letter: l,
          opened: false
        });
      });
      this.letterScore = this.letters.length * 100;
      this.currentQuestion.asked = true;
    },
    giveLetter() {
      let randomLetterIndex = Math.floor(Math.random() * this.letters.length);

      if (this.letterScore <= 100) {
        this.disabledGiveLetterBtn = true;
        return;
      }

      let letter = this.letters[randomLetterIndex];

      while (letter.opened) {
        randomLetterIndex = Math.floor(Math.random() * this.letters.length);
        letter = this.letters[randomLetterIndex];
      }

      letter.opened = true;
      this.letterScore -= 100;
    },
    answer() {
      this.disabledAnswerBtn = true;
      if (
        this.competitorAnswer ===
        this.currentQuestion.answer.toLocaleUpperCase("tr")
      ) {
        this.score += this.letterScore;
        this.showMessage("Tebrikler, doğru cevap", "success");
      } else {
        this.score -= this.letterScore;
        this.showMessage("Yanlış cevap", "error");
      }

      setTimeout(() => {
        this.disabledAnswerBtn = false;
        this.giveQuestion();
      }, 4000);
    },
    finish() {
      clearInterval(this.time);
      this.currentQuestion = null;
      this.completed = true;
    },
    showMessage(text, type) {
      this.message = text;
      if (type === "error") {
        this.messageClass = "bg-danger text-white";
        this.showLetters();
      } else if (type === "success") {
        this.messageClass = "bg-success text-white";
        this.showLetters();
      } else {
        this.messageClass = "bg-dark text-white";
      }

      if (this.messageTime) {
        clearTimeout(this.messageTime);
        this.messageTime = null;
      }

      this.messageTime = setTimeout(() => {
        this.message = null;
      }, 4000);
    },
    showLetters() {
      this.letters.map(l => (l.opened = true));
    }
  }
};
</script>