<template>
  <div class="card mb-2">
    <div @click="moveToPage(i.id)" class="card-body p-2 d-flex" v-for="i in todos" :key="i.id" style="cursor:pointer">
      <div class="form-check flex-grow-1">
        <label class="form-check-label" :class="{ todo: i.completed }">
          {{ i.subject }}
          <input type="checkbox" class="form-check-input" :value="todos.completed" @change="toggleTodo(i.id, $event)"
            @click.stop :checked="i.completed" />

        </label>
      </div>
      <div>
        <button class="btn btn-danger btn-sm" @click.stop="openModal(i.id)">
          삭제
        </button>
        <!-- deleteTodo(i.id) Modal.vue에서 삭제하기 버튼 클릭시 호출 -->
      </div>

    </div>
  </div>
  <Modal v-if="showModal" @close="closeModal" @delete="deleteTodo" />
</template>

<script>
import { ref, Ref } from "vue";
import { useRouter } from "vue-router";
import Modal from "./Modal.vue";
export default {
  props: {
    todos: {
      type: Array,
      required: true,
    },
  },
  components: {
    Modal
  },
  emits: ["toggle-todo", "delete-todo"],

  setup(props, { emit }) {
    const router = useRouter();
    let showModal = ref(false);
    let todoDeleteId = ref(null);
    const openModal = (id) => {
      console.log("지울", id);
      todoDeleteId.value = id;
      showModal.value = true;
    }
    const closeModal = () => {
      todoDeleteId.value = null;
      showModal.value = false;
    }

    const toggleTodo = (index, event) => {
      emit("toggle-todo", index, event.target.checked);
    };
    const deleteTodo = (index) => {
      emit("delete-todo", index);
    };
    const moveToPage = (todoId) => {
      // router.push("/todos/"+todoId);
      router.push({
        name: "Todo",
        params: {
          id: todoId,
        }
      })
    }
    return {
      toggleTodo,
      deleteTodo,
      moveToPage,
      showModal,
      openModal,
      closeModal
    };
  },
};
</script>

<style>
.form-check-label {
  cursor: pointer;
}
</style>
