<template>
  <div class="container">
    <h2>Quản lý công việc</h2>
    <div class="add-task">
      <input
        type="text"
        placeholder="Nhập tên công việc"
        v-model="inputValue"
      />
      <button @click="addTask">Thêm công việc</button>
    </div>
    <div class="filters">
      <button :class="{ active: isActiveAll }" @click="allJob">Tất cả</button>
      <button :class="{ active: isActiveComplete }" @click="completeJob">
        Hoàn thành
      </button>
      <button :class="{ active: isActiveContinue }" @click="continueJob">
        Đang thực hiện
      </button>
    </div>
    <div v-if="isLoad" class="task-list">
      <div class="task" v-for="(task, index) in jobs" :key="index">
        <input
          type="checkbox"
          v-model="task.status"
          @change="updateStatus(task)"
        />
        <span v-if="task.status"
          ><s>{{ task.nameJob }}</s></span
        >
        <span v-else>{{ task.nameJob }}</span>
        <div class="actions">
          <span @click="editNameJob(task.id)" class="edit">✎</span>
          <span class="delete" @click="deleteTask(task.id)">🗑</span>
        </div>
      </div>
    </div>
    <div v-else class="task-list">
      <div class="task" v-for="(task, index) in listJob" :key="index">
        <input
          type="checkbox"
          v-model="task.status"
          @change="updateStatus(task)"
        />
        <span v-if="task.status"
          ><s>{{ task.nameJob }}</s></span
        >
        <span v-else>{{ task.nameJob }}</span>
        <div class="actions">
          <span class="edit">✎</span>
          <span class="delete" @click="deleteTask(task.id)">🗑</span>
        </div>
      </div>
    </div>
    <div class="clear-buttons">
      <button class="clear-completed" @click="deleteAllComplete">
        Xóa công việc hoàn thành
      </button>
      <button class="clear-all" @click="deleteAll">Xóa tất cả công việc</button>
    </div>
    <div v-show="isOpen" class="modal">
      <div class="modal-content">
        <span class="close" @click="closeEditModal">&times;</span>
        <h3>Sửa công việc</h3>
        <input type="text" v-model="editJob" />
        <button @click="saveEdit">Lưu</button>
      </div>
    </div>
  </div>
</template>
<script setup>
import axios from "axios";
import { onMounted, ref } from "vue";
const listJob = ref([]);
const jobs = ref(listJob.value);
const inputValue = ref("");
const editJob = ref("");
const isOpen = ref(false);
const isLoad = ref(false);
const isActiveAll = ref(true);
const isActiveComplete = ref(false);
const isActiveContinue = ref(false);
const idJob = ref(null);
const getAllJobs = async () => {
  const res = await axios.get("http://localhost:3000/jobs");
  listJob.value = res.data;
};
onMounted(() => {
  getAllJobs();
});
const addTask = async () => {
  const newJob = {
    id: listJob.value.length + 1,
    nameJob: inputValue.value,
    status: false,
  };
  const res = await axios.post("http://localhost:3000/jobs", newJob);
  getAllJobs();
};
const deleteTask = async (id) => {
  const res = await axios.delete(`http://localhost:3000/jobs/${parseInt(id)}`);
  getAllJobs();
};
const updateStatus = async (task) => {
  await axios.put(`http://localhost:3000/jobs/${task.id}`, task);
  getAllJobs();
};
const completeJob = () => {
  jobs.value = listJob.value.filter((job) => job.status);
  isActiveAll.value = false;
  isActiveComplete.value = true;
  isActiveContinue.value = false;
  isLoad.value = true;
};

const continueJob = () => {
  jobs.value = listJob.value.filter((job) => !job.status);
  isActiveAll.value = false;
  isActiveComplete.value = false;
  isActiveContinue.value = true;
  isLoad.value = true;
};

const allJob = () => {
  jobs.value = [...listJob.value];
  isActiveAll.value = true;
  isActiveComplete.value = false;
  isActiveContinue.value = false;
  isLoad.value = true;
};
const deleteAllComplete = async () => {
  await axios.delete("http://localhost:3000/jobs?status=true");
  getAllJobs();
};

const deleteAll = async () => {
  await axios.delete("http://localhost:3000/jobs");
  getAllJobs();
};
const editNameJob = (id) => {
  isOpen.value = true;
  idJob.value = id;
  const jobToEdit = listJob.value.find(job => job.id === id);
  if (jobToEdit) {
    editJob.value = jobToEdit.nameJob; 
  }
}

const saveEdit = async() => {
  const jobToUpdate = listJob.value.find(job => job.id === idJob.value);
  if (jobToUpdate) {
    const updateJob = {
      ...jobToUpdate,
      nameJob: editJob.value
    };
    await axios.put(`http://localhost:3000/jobs/${idJob.value}`, updateJob);
    closeEditModal();
    await getAllJobs(); 
  }
}

const closeEditModal = () => {
  isOpen.value = false;
}
</script>

<style scoped>
.modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  z-index: 1000;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
  width: 300px;
}

.close {
  cursor: pointer;
  float: right;
}
.container {
  font-family: Arial, sans-serif;
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
}

h2 {
  text-align: center;
  color: #333;
}

.add-task {
  display: flex;
  margin-bottom: 20px;
}

input[type="text"] {
  flex-grow: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px 0 0 4px;
}

button {
  padding: 10px 15px;
  background-color: #4285f4;
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
}

.filters {
  display: flex;
  margin-bottom: 20px;
}

.filters button {
  flex-grow: 1;
  background-color: #f1f3f4;
  color: #333;
  border: 1px solid #ddd;
  border-radius: 0;
}

.filters button.active {
  background-color: #4285f4;
  color: white;
}

.task {
  display: flex;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}

.task input[type="checkbox"] {
  margin-right: 10px;
}

.task span {
  flex-grow: 1;
}

.task span.completed {
  text-decoration: line-through;
  color: #888;
}

.actions {
  display: flex;
  gap: 10px;
}

.actions span {
  cursor: pointer;
}

.clear-buttons {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}

.clear-buttons button {
  background-color: #f44336;
  border-radius: 4px;
}
</style>
