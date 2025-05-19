<template>
  <v-container fluid class="fill-height pa-4">
    <v-row justify="center" align="center">
      <v-col cols="12" md="8" lg="6" class="text-center">
        <transition name="fade" mode="out-in">
          <div v-if="page === 'start'">
            <h2 class="mb-4">เครื่องดื่มที่เหมาะกับเราจะเป็นแบบไหนกันนะ?</h2>
            <v-btn color="primary" rounded="pill" @click="page = 'intro'">ถัดไป</v-btn>
          </div>
          <div v-else-if="page === 'intro'">
            <h2 class="mb-4">ลองมาทำแบบทดสอบกับเราดูไหม?</h2>
            <v-btn color="success" rounded="pill" @click="page = 'quiz'">เริ่มทำแบบทดสอบ</v-btn>
          </div>
          <div v-else-if="page === 'quiz'">
            <h3 class="mb-4">{{ questions[currentQuestionIndex].question }}</h3>
            <div
              v-for="(option, index) in questions[currentQuestionIndex].options"
              :key="index"
              class="mb-2"
            >
              <v-btn
                block
                rounded="pill"
                variant="outlined"
                :class="{ 'active-choice': selectedAnswer === option.value }"
                @click="selectAnswer(questions[currentQuestionIndex].value, option.value)"
              >
                {{ option.text }}
              </v-btn>
            </div>
            <div class="mt-6">
              <v-btn
                v-if="currentQuestionIndex < questions.length - 1"
                color="primary"
                rounded="pill"
                @click="nextQuestion"
                :disabled="!selectedAnswer"
              >
                ถัดไป
              </v-btn>
              <v-btn
                v-else
                color="success"
                rounded="pill"
                @click="calculateResult"
                :disabled="userAnswers.length < questions.length || !selectedAnswer"
              >
                ดูผลลัพธ์
              </v-btn>
            </div>
          </div>
          <div v-else-if="page === 'result'">
            <h2>ค็อกเทลที่เหมาะกับคุณคือ...</h2>
            <div v-if="loading">
              <v-progress-circular indeterminate color="primary"></v-progress-circular>
            </div>
            <div v-else-if="cocktailResult">
              <h3>{{ cocktailResult.strDrink }}</h3>
              <v-img
                :src="cocktailResult.strDrinkThumb"
                max-height="300"
                contain
                class="mb-4"
              ></v-img>
              <!--  -->
              <p><b>รายละเอียด</b>: {{ personalityDescription }}</p>
              <p><b>สรุป</b>:</p>
              <ul>
                <li v-for="(item, index) in personalitySummary" :key="index">{{ item }}</li>
              </ul>
              <!--  -->
            </div>
            <div v-else-if="error">
              <p class="error--text">{{ errorMessage }}</p>
            </div>
            <v-btn class="mt-4" rounded="pill" @click="resetQuiz">ทำแบบทดสอบอีกครั้ง</v-btn>
          </div>
        </transition>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, computed } from 'vue';
import axios from 'axios';

const page = ref('start');
const currentQuestionIndex = ref(0);
const userAnswers = ref([]);
const answerCounts = ref({ a: 0, b: 0, c: 0 });
const recommendedCocktail = ref('');
const cocktailResult = ref(null);
const loading = ref(false);
const error = ref(false);
const errorMessage = ref('');
const selectedAnswer = ref(null);

const questions = ref([
  {
    question: 'ถ้ามีวันหยุดยาว คุณอยากจะใช้เวลาทำอะไร?',
    options: [
      { text: 'ไปทะเล นอนรับลม จิบเครื่องดื่มเย็น ๆ', value: 'a' },
      { text: 'ไปปาร์ตี้กับเพื่อนที่ต่างจังหวัด จัดเต็มให้สุด', value: 'b' },
      { text: 'อยู่บ้าน อ่านหนังสือหรือดูหนังดี ๆ สักเรื่อง', value: 'c' },
    ],
    value: 'preference1',
  },
  {
    question: 'ถ้าเพื่อนมีปัญหาและมาปรึกษาคุณ คุณจะ......',
    options: [
      { text: 'ชวนออกไปข้างนอกเปลี่ยนบรรยากาศ', value: 'a' },
      { text: 'พูดตรง ๆ ให้กำลังใจแบบไม่อ้อมค้อม', value: 'b' },
      { text: 'นั่งฟังอย่างตั้งใจ พร้อมช่วยคิดหาทางออกอย่างรอบคอบ', value: 'c' },
    ],
    value: 'preference2',
  },
  {
    question: 'ถ้ามีคนใหม่ ๆ เข้ามาในกลุ่ม คุณมักจะ......',
    options: [
      { text: 'ทักก่อน ยิ้มแย้ม สร้างบรรยากาศ', value: 'a' },
      { text: 'สังเกตครู่นึง แล้วเริ่มแสดงความเป็นตัวเองแบบชัดเจน', value: 'b' },
      { text: 'นั่งฟังมากกว่าพูด แต่ออกความเห็นแบบมีน้ำหนักเมื่อจำเป็น', value: 'c' },
    ],
    value: 'preference3',
  },
]);

const personalityDescriptions = ref({
  mojito: 'คุณเป็นคนสดใส รักอิสระ และชอบกิจกรรมที่ทำให้รู้สึกกระปรี้กระเปร่า เหมือนกับความสดชื่นของ Mojito ที่มีรสชาติของมิ้นท์และมะนาว',
  margarita: 'คุณเป็นคนชอบเข้าสังคม มีความมั่นใจ และพร้อมที่จะสนุกสนานไปกับเพื่อนฝูง เหมือนกับ Margarita ที่มีรสชาติจัดจ้านและเป็นที่นิยมในงานปาร์ตี้',
  'whiskey sour': 'คุณเป็นคนสุขุม รอบคอบ และให้ความสำคัญกับความสมดุล เหมือนกับ Whiskey Sour ที่มีรสชาติเปรี้ยวอมหวานอย่างลงตัว',
});

const personalitySummaries = ref({
  mojito: ['รักความสดชื่น', 'ชอบกิจกรรม', 'เปิดเผย'],
  margarita: ['ชอบปาร์ตี้', 'มั่นใจในตัวเอง', 'เข้ากับคนง่าย'],
  'whiskey sour': ['สุขุม', 'รอบคอบ', 'ชอบความสมดุล'],
});

const selectAnswer = (questionValue, answerValue) => {
  userAnswers.value[currentQuestionIndex.value] = answerValue;
  selectedAnswer.value = answerValue;
};

const nextQuestion = () => {
  if (currentQuestionIndex.value < questions.value.length - 1 && selectedAnswer.value) {
    currentQuestionIndex.value++;
    selectedAnswer.value = null; // รีเซ็ต selectedAnswer สำหรับคำถามใหม่
  }
};
//คำนวณ logic การเลือกช้อยส์
const calculateResult = () => {
  if (userAnswers.value.length === questions.value.length) {
    answerCounts.value = { a: 0, b: 0, c: 0 };
    userAnswers.value.forEach(answer => {
      answerCounts.value[answer]++;
    });

    if (answerCounts.value.a > answerCounts.value.b && answerCounts.value.a > answerCounts.value.c) {
      recommendedCocktail.value = 'mojito';
    } else if (answerCounts.value.b > answerCounts.value.a && answerCounts.value.b > answerCounts.value.c) {
      recommendedCocktail.value = 'margarita';
    } else {
      recommendedCocktail.value = 'whiskey sour';
    }

    fetchCocktailDetails();
    page.value = 'result';
  }
};

//เรียก api จาก free cocktail
const fetchCocktailDetails = async () => {
  loading.value = true;
  error.value = false;
  cocktailResult.value = null;

  try {
    const response = await axios.get(
      `https://www.thecocktaildb.com/api/json/v1/1/search.php?s=${recommendedCocktail.value}`
    );
    loading.value = false;
    if (response.data && response.data.drinks && response.data.drinks.length > 0) {
      cocktailResult.value = response.data.drinks[0];
    } else {
      error.value = true;
      errorMessage.value = `ไม่พบข้อมูลค็อกเทล "${recommendedCocktail.value}"`;
    }
  } catch (err) {
    loading.value = false;
    error.value = true;
    errorMessage.value = 'เกิดข้อผิดพลาดในการเรียก API';
    console.error('Error fetching cocktail:', err);
  }
};

const personalityDescription = computed(() => {
  return personalityDescriptions.value[recommendedCocktail.value] || 'ไม่พบคำอธิบาย';
});

const personalitySummary = computed(() => {
  return personalitySummaries.value[recommendedCocktail.value] || [];
});

const resetQuiz = () => {
  page.value = 'start';
  currentQuestionIndex.value = 0;
  userAnswers.value = [];
  answerCounts.value = { a: 0, b: 0, c: 0 };
  recommendedCocktail.value = '';
  cocktailResult.value = null;
  loading.value = false;
  error.value = false;
  errorMessage.value = '';
  selectedAnswer.value = null;
};
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.active-choice {
  background-color: #e0f2f7 !important;
  border-color: #26a69a !important;
}
</style>