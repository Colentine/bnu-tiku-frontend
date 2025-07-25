<template>
  <div class="upload-view">
    <a-breadcrumb :style="{ margin: '16px 0', marginLeft: '20px' }">
      <a-breadcrumb-item>智慧题库</a-breadcrumb-item>
      <a-breadcrumb-item>试题篮</a-breadcrumb-item> </a-breadcrumb
    ><br />
  </div>
  <div class="question-bucket">
    <NeoQuestionCard
      v-if="allQuestions.length > 0"
      :loading="false"
      :show-add-to-basket="false"
      :page-type="'bucket'"
      :all-questions="allQuestions"
    />
    <div class="basket-content" v-else>试题篮为空，快去挑选试题吧</div>
  </div>
  <div class="basket-footer">
    <div class="basket-info">
      已选 {{ allQuestions.length }} / {{ maxCount }} 道题
    </div>
    <div class="basket-actions">
      <a-button class="clear-button" type="primary" ghost @click="clearAll"
        >清空</a-button
      >
      <a-button
        class="download-word-button"
        type="primary"
        @click="exportAsWord"
        :loading="downloading"
        >导出为 Word</a-button
      >
      <!--      <a-button-->
      <!--        class="download-pdf-button"-->
      <!--        type="primary"-->
      <!--        @click="$emit('export', 'pdf')"-->
      <!--        >导出为 PDF</a-button-->
      <!--      >-->
    </div>
  </div>
</template>

<script setup lang="ts" async>
import QuestionCard from "@/components/QuestionCard.vue";
import { onBeforeMount, onMounted, ref, toRefs } from "vue";
import { useQuestionBasketStore } from "@/stores/questionBasket";
import { message } from "ant-design-vue";
import { exportQuestionByIds, getQuestionsByIds } from "@/api/question";
import { storeToRefs } from "pinia";
import { useAllQuestionsStore } from "@/stores/AllQuestions";
import { syncUserFromServer } from "@/api/user";
import NeoQuestionCard from "@/components/NeoQuestionCard.vue";
const allQuestionIds = ref<number[]>([]);
const allQuestions = ref<any>([]);
const maxCount = ref(20);
const { questionBasket } = toRefs(useQuestionBasketStore());
const downloading = ref(false);
allQuestionIds.value = Array.from(questionBasket.value.values());
onMounted(async () => {
  const res = await getQuestionsByIds(allQuestionIds.value);
  allQuestions.value = res.data.data;
  allQuestions.value.map((q) => {
    q.stem = q.stem.replace(/\n/g, "<br>");
    q.question_answer = q.question_answer.replace(/\n/g, "<br>");
    q.question_explanation = q.question_explanation.replace(/\n/g, "<br>");
  });
  console.log("getQuestionsByIds", allQuestions.value);
});

console.log("bucket:", allQuestions.value);
const clearAll = () => {
  questionBasket.value.clear();
  allQuestions.value = [];
  message.success("试题篮已清空");
};

const exportAsWord = async () => {
  downloading.value = true;
  const allQuestionIds = allQuestions.value.map(
    (question: any) => question.question_id
  );
  exportQuestionByIds(allQuestionIds, "docx")
    .then((res) => {
      const filename = `题库导出.doc`;
      const blob = new Blob([res.data]);
      const url = window.URL.createObjectURL(blob);
      // 模拟 a 标签点击下载
      const a = document.createElement("a");
      a.href = url;
      a.download = filename;
      document.body.appendChild(a);
      a.click();
      a.remove();
      window.URL.revokeObjectURL(url);
    })
    .finally(() => {
      clearAll();
      downloading.value = false;
    });
};
</script>

<style scoped>
.question-bucket {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 100px;
}
.basket-footer {
  position: fixed;
  bottom: 0;
  left: 25%;
  right: 25%;
  width: 50%;
  z-index: 100;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 20px;
  background-color: #fafafa;
  border-top: 1px solid #eee;
  box-shadow: 0 -1px 4px rgba(0, 0, 0, 0.05);
  border-radius: 10px 10px 0 0;
}

.basket-info {
  font-size: 17px;
  color: #666;
}
.basket-actions a-button + a-button {
  margin-left: 12px;
}
.clear-button {
  margin-right: 20px;
  font-size: 17px;
  height: 40px;
  background-color: #ff0000;
  border-color: #ff0000;
  color: #ffffff;
  box-shadow: none;
}
.clear-button:hover {
  color: #0e0a0a;
}

.download-pdf-button {
  font-size: 17px;
  height: 40px;
  color: #1677ff;
  border-color: #1677ff;
  background-color: white;
  box-shadow: none;
}
.download-pdf-button:hover {
  color: #ffffff;
}

.download-word-button {
  margin-right: 20px;
  font-size: 17px;
  height: 40px;
  box-shadow: none;
}

.download-word-button:hover {
  color: #ffffff;
}
.basket-content {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 25px;
  height: 50vh;
}
</style>
