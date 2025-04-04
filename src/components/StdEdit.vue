<template>
  <div class="edit-container">
    <form @submit.prevent="save">
      <div v-if="!isCourseForm">
        <div class="input-group">
          <label>Name:</label>
          <input v-model="form.name" required />
        </div>
        <div class="input-group">
          <label>Student ID:</label>
          <input v-model="form.student_id" required />
        </div>
        <div class="input-group">
          <label>Major:</label>
          <input v-model="form.major" required />
        </div>
        <div class="input-group">
          <label>School:</label>
          <input v-model="form.school" required />
        </div>
      </div>
      <div v-else>
        <div class="input-group">
          <label>Course Code:</label>
          <input v-model="form.code" required />
        </div>
        <div class="input-group">
          <label>Course Name:</label>
          <input v-model="form.name" required />
        </div>
        <div class="input-group">
          <label>Credits:</label>
          <input v-model.number="form.credits" type="number" min="0" required />
        </div>
        <div class="input-group">
          <label>Grade:</label>
          <input v-model="form.grade" required />
        </div>
      </div>
      <div class="button-group">
        <button type="submit">Save</button>
        <button @click="cancel">Cancel</button>
        <button v-if="isCourseForm && form.code" @click="deleteCourse">Delete</button>
      </div>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  props: ['student', 'isCourseForm'],
  data() {
    return {
      form: { ...this.student },
    };
  },
  methods: {
    save() {
      if (this.isCourseForm) {
        this.$emit('saveCourse', this.form);
      } else {
        this.$emit('save', this.form);
      }
    },
    cancel() {
      this.$emit('cancel');
    },
    deleteCourse() {
      axios
        .delete(`http://localhost:3001/students/1/courses/${this.form.code}`)
        .then(() => {
          this.$emit('cancel');
        })
        .catch(error => {
          console.error('Error deleting course:', error);
        });
    },
  },
};
</script>


<style scoped>
.edit-container {
  margin: 20px;
  background-color: rgba(70, 60, 50, 0.7);
  padding: 20px;
  border-radius: 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  font-family: 'Arial', sans-serif;
  color: #e0e0e0;
}

form {
  display: flex;
  flex-direction: column;
}

.input-group {
  margin-bottom: 16px;
  display: flex;
  flex-direction: column;
}

input {
  padding: 10px;
  font-size: 1rem;
  border-radius: 8px;
  border: 1px solid rgba(100, 90, 80, 0.7);
  outline: none;
  transition: border-color 0.3s;
  background-color: rgba(80, 70, 60, 0.7);
  color: #e0e0e0;
}

input:focus {
  border-color: #ffcc80;
}

button {
  padding: 10px 20px;
  background-color: #ffcc80;
  color: #333;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #ffb300;
}

.button-group {
  display: flex;
  gap: 10px;
}
</style>