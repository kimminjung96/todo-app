<template>
  <h1 v-if="editing">일정 상세 페이지</h1>
  <div v-if="loading">Loading...</div>
  <form v-else @submit.prevent="onSave">
    <div class="row">
      <div class="col-6">
        <div class="form-group">
          <label>일정명: </label>
          <div class="red" v-if="subjectError">{{ subjectError }}</div>
          <input type="text" class="form-control" v-model="todo.subject">
        </div>
      </div>
      <div v-if="editing" class="col-6">
        <div class="form-group">
          <label>수행상태: </label>
          <button class="btn" :class="(todo.completed) ? ('btn-success') : ('btn-danger')" type="button"
            @click="toggleTodoStatus">
            {{ (todo.completed) ? ("완료") : ("미완료") }}
          </button>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-12">
        <div class="form-group">
          <label>일정내용 :</label>
          <textarea class="form-control" cols="30" rows="10" v-model="todo.body"></textarea>
        </div>
      </div>
    </div>
    <button class="btn btn-primary" :disabled="!todoUpdate">{{ editing ? "저장" : "등록" }}</button>
    <button class="btn btn-outline-dark ms-2" @click="moveToTodoListPage">취소</button>
  </form>
  <transition name="fade">
    <Toast v-if="showToast" :message="toastMessage" :type="toastAlertType" />
  </transition>
</template>
<script>
import { useRoute, useRouter } from "vue-router";
import axios from "axios";
import { ref, computed, onUnmounted } from "vue";
import _ from "lodash";
import Toast from "@/components/Toast.vue";

export default {
  props: {
    editing: { type: Boolean, default: false }
  },
  components: {
    Toast
  },
  setup(props) {
    const subjectError = ref(null);
    const originalTodo = ref(null);
    const route = useRoute();
    const routers = useRouter();
    const todo = ref({
      subject: "",
      completed: false,
      body: "",
    });
    const loading = ref(false);
    const todoId = route.params.id
    const url = "http://localhost:8080/todos/"

    const showToast = ref(false);
    const toastMessage = ref('');
    const toastAlertType = ref('');
    const timeout = ref(null);

    onUnmounted(() => { clearTimeout(timeout.value) })

    const triggerToast = (msg, type = 'info') => {
      toastMessage.value = msg;
      toastAlertType.value = type;
      showToast.value = true;
      timeout.value = setTimeout(() => {
        toastMessage.value = '';
        toastAlertType.value = '';
        showToast.value = false;
      }, 5000)
    }

    const todoUpdate = computed(() => {
      return !_.isEqual(todo.value, originalTodo.value)
    })
    const toggleTodoStatus = () => {
      todo.value.completed = !todo.value.completed;
    }
    const onSave = () => {
      let response;
      const data = {
        subject: todo.value.subject,
        completed: todo.value.completed,
        body: todo.value.body,
      }

      subjectError.value = ""
      if (!todo.value.subject) {
        subjectError.value = "일정명은 필수사항 입니다."
      }
      if (props.editing) {
        //editing 일때 =>기존 일정업데이트
        axios.put(`${url}${todoId}`, data).then((res) => {
          originalTodo.value = { ...res.data }
          triggerToast("등록이 완료되었습니다", "info");
        }).catch((err) => {
          console.error(err)
          triggerToast("일시적으로 오류가 발생하였습니다. 잠시후 다시 이용 해주세요.", "danger")
        })

      } else {
        //editing 아닐때 => create
        axios.post(`${url}`, data).then((res) => {
          response = res;
          console.log("response😀😀", response);
          triggerToast("등록이 완료되었습니다", "info");
          //추가등록후 빈값으로 만들기
          todo.value.subject = "";
          todo.value.body = "";
        }).catch((err) => {
          console.error(err)
          triggerToast("일시적으로 오류가 발생하였습니다. 잠시후 다시 이용 해주세요.", "danger")
        })
      }
    }/* //onSave */

    const getTodo = () => {
      loading.value = true;
      axios.get(`${url}${todoId}`).then((res) => {
        todo.value = { ...res.data };
        originalTodo.value = { ...res.data };
        loading.value = false;
      }).catch((err) => { console.error(err); loading.value = false; })
    }
    if (props.editing) {
      getTodo();
    }
    const moveToTodoListPage = () => {
      routers.push({
        name: "Todos"
      })
    }
    return {
      subjectError,
      onSave, toggleTodoStatus, triggerToast,
      todo, toastAlertType,
      loading,
      moveToTodoListPage,
      todoUpdate, showToast, toastMessage
    }
  }
}
</script>
<style scoped>
/* scoped 이 컴포넌트안에서만 쓰겟다 설정 */
/* 전역으로 쓰임 */
.red {
  color: red;
}

.fade-enter-active,
.fade-leave-active {
  transition: all 1s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-30px);
}

.fade-enter-to,
.fade-leave-from {
  opacity: 1;
  transform: translateY(0px);
}
</style>