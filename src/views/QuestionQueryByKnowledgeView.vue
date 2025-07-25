<template>
  <div class="all-content">
    <a-breadcrumb :style="{ margin: '16px 0', marginLeft: '20px' }">
      <a-breadcrumb-item>智慧题库</a-breadcrumb-item>
      <a-breadcrumb-item>知识点选题</a-breadcrumb-item>
    </a-breadcrumb>
    <div class="knowledge-content">
      <a-row :gutter="[12, 12]" justify="center" style="flex-wrap: nowrap">
        <a-col style="width: 300px">
          <KnowledgePointTree @send="receiveData" />
        </a-col>
        <a-col style="padding-left: 32px; width: 1000px">
          <a-row
            ><a-col flex="120px">
              共计 <span style="color: #2877ff">{{ total }}</span> 道试题
            </a-col>
            <a-col flex="1" />
            <a-col flex="1">
              <div style="text-align: right" v-if="selectedKp != 'beforeMount'">
                正在搜索的知识点：<span style="color: #2877ff">{{
                  selectedKp
                }}</span>
              </div>
              <div style="text-align: right" v-else>
                正在搜索的知识点：<span style="color: #2877ff"> 全部 </span>
              </div></a-col
            >
          </a-row>
          <a-row>
            <QuestionFilter @update-filter="receiveLabel" />
          </a-row>
          <a-row :gutter="[12, 12]">
            <QuestionCard
              :all-questions="allQuestions"
              :loading="loading"
              :show-add-to-basket="true"
            />
          </a-row>
          <a-col
            style="
              display: flex;
              justify-content: center;
              width: 1000px;
              margin-top: 20px;
            "
          >
            <a-pagination
              show-quick-jumper
              v-model:current="currentPageNumber"
              :total="total"
              v-model:page-size="currentPageSize"
              @change="onChange"
              style="margin-bottom: 30px"
          /></a-col>
        </a-col>
      </a-row>
    </div>
  </div>
</template>
<script setup lang="ts">
import KnowledgePointTree from "@/components/KnowledgePointTree.vue";
import { onMounted, ref } from "vue";
import QuestionCard from "@/components/QuestionCard.vue";
import { getQuestionByCombination, QueryParams } from "@/api/question";
import QuestionFilter from "@/components/QuestionFilter.vue";
import { useAllQuestionsStore } from "@/stores/AllQuestions";
import { storeToRefs } from "pinia";
import { useFilterStore } from "@/stores/filter";
const allQuestionsStore = useAllQuestionsStore();
const { allQuestions } = storeToRefs(allQuestionsStore);
const currentPageNumber = ref<number>(1);
const currentPageSize = ref<number>(10);
const selectedKp = ref("beforeMount");
const combinationLabel = ref<QueryParams>({
  knowledgePointName: "",
  keyword: "",
  difficulty: "all",
  gradeId: -1,
  simpleQuestionType: -1,
  pageNumber: 1,
  pageSize: 10,
});
const total = ref(0);
const loading = ref(true);

// 模拟“加入试题篮”操作
const filterStore = useFilterStore();
const { selected, label } = storeToRefs(filterStore);
onMounted(() => {
  combinationLabel.value.gradeId = label.value.grade;
  combinationLabel.value.difficulty = label.value.difficulty;
  combinationLabel.value.simpleQuestionType = label.value.type;
  getQuestionByCombination(combinationLabel.value)
    .then((res) => {
      allQuestions.value = res.data.data.questions;
      allQuestions.value.map((q) => {
        q.stem = q.stem.replace(/\n/g, "<br>");
        q.question_answer = q.question_answer.replace(/\n/g, "<br>");
        q.question_explanation = q.question_explanation.replace(/\n/g, "<br>");
      });
      total.value = res.data.data.totalCount;
    })
    .finally(() => {
      loading.value = false;
    });
});

const receiveData = (data: any) => {
  if (data.resultData != null && data.selectedKey != null) {
    allQuestions.value = data.resultData.questions;
    allQuestions.value.map((q) => {
      q.stem = q.stem.replace(/\n/g, "<br>");
      q.question_answer = q.question_answer.replace(/\n/g, "<br>");
      q.question_explanation = q.question_explanation.replace(/\n/g, "<br>");
    });
    selectedKp.value = data.selectedKey;
    total.value = data.resultData.totalCount;
  }
  currentPageNumber.value = 1;
  currentPageSize.value = 10;
  loading.value = data.loading;
};
const receiveLabel = async (data: any) => {
  loading.value = true;
  combinationLabel.value.knowledgePointName = selectedKp.value;
  combinationLabel.value.gradeId = data.selected.grade;
  combinationLabel.value.difficulty = data.selected.difficulty;
  combinationLabel.value.simpleQuestionType = data.selected.type;
  label.value.grade = data.selected.grade;
  label.value.difficulty = data.selected.difficulty;
  label.value.type = data.selected.type;
  currentPageSize.value = 10;
  currentPageNumber.value = 1;
  combinationLabel.value.pageNumber = currentPageNumber.value;
  combinationLabel.value.pageSize = currentPageSize.value;
  console.log(combinationLabel.value);

  const res = await getQuestionByCombination(combinationLabel.value);

  allQuestions.value = res.data.data.questions;
  allQuestions.value.map((q) => {
    q.stem = q.stem.replace(/\n/g, "<br>");
    q.question_answer = q.question_answer.replace(/\n/g, "<br>");
    q.question_explanation = q.question_explanation.replace(/\n/g, "<br>");
  });
  total.value = res.data.data.totalCount;
  loading.value = false;
  // alert(combinationLabel.value.grade);
  // message.success("**********");
};

// const onChange = async (pageNumber: number) => {
//   loading.value = true;
//   const res = await getQuestionsByKp(
//     selectedKp.value,
//     pageNumber.toString(),
//     currentPageSize.value.toString()
//   );
//   allQuestions.value = res.data.questions;
//   total.value = res.data.totalCount;
//   loading.value = false;
//   console.log(allQuestions.value);
// };

const onChange = async (pageNumber: number) => {
  loading.value = true;
  combinationLabel.value.knowledgePointName = selectedKp.value;
  combinationLabel.value.pageNumber = pageNumber;
  combinationLabel.value.pageSize = currentPageSize.value;

  const res = await getQuestionByCombination(combinationLabel.value);

  allQuestions.value = res.data.data.questions;
  allQuestions.value.map((q) => {
    q.stem = q.stem.replace(/\n/g, "<br>");
    q.question_answer = q.question_answer.replace(/\n/g, "<br>");
    q.question_explanation = q.question_explanation.replace(/\n/g, "<br>");
  });
  total.value = res.data.data.totalCount;
  loading.value = false;
  console.log(allQuestions.value);
};
</script>

<style scoped>
.knowledge-content {
  display: flex;
  justify-content: center;
}
</style>
