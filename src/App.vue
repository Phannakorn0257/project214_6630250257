<template>
  <div class="app-container">
    <video autoplay muted loop class="background-video" playsinline>
      <source src="/public/img/backgroundprofile.mp4" type="video/mp4" />
      Your browser does does not support the video tag>
    </video>
    <div class="content">
      <h1 class="title">Student Information</h1>
      <div class="student-container">
        <div class="profile-image-container">
          <img :src="profileImages[currentImageIndex]" alt="Profile Picture" class="profile-img" />
        </div>
        <div v-if="student" class="student-info">
          <h2>{{ student.name }}</h2>
          <p>Student ID: {{ student.student_id }}</p>
          <p>Major: {{ student.major }}</p>
          <p>School: {{ student.school }}</p>
        </div>
        <button v-if="student" class="edit-button" @click="editStudent()">Edit Student Info</button>
        <hr class="divider" />
        <h3>Courses</h3>
        <button v-if="student" class="add-course-button" @click="addCourse()">Add Course</button>
        <ul v-if="student" class="course-list">
          <li v-for="course in student.courses" :key="course.code" class="course-item">
            <span>{{ course.code }} - {{ course.name }} - {{ course.credits }} Credits - Grade: {{ course.grade }}</span>
            <div class="action-buttons">
              <div class="button-column">
                <button class="edit-course-button" @click="editCourse(course)">Edit</button>
                <button class="delete-course-button" @click="deleteCourse(course)">Delete</button>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <StdEdit
      v-if="editStudentForm || editCourseForm"
      :student="editStudentForm ? student : selectedCourse"
      :isCourseForm="editCourseForm"
      @save="saveStudent"
      @saveCourse="saveCourse"
      @cancel="cancelEdit"
      ref="editForm"
    />
  </div>
</template>

<script>
import StdEdit from './components/StdEdit.vue';
import axios from 'axios';

export default {
  components: {
    StdEdit,
  },
  data() {
    return {
      student: null,
      editStudentForm: false,
      editCourseForm: false,
      selectedCourse: null,
      profileImages: [
        '/public/img/phannakorn.jpg',
        '/public/img/phannakorn3.jpg',
      ],
      currentImageIndex: 0,
    };
  },
  mounted() {
    this.fetchStudentData();
    this.startImageRotation();
  },
  methods: {
    fetchStudentData() {
      axios.get('http://localhost:3001/students/1').then((response) => {
        this.student = response.data;
      });
    },
    editStudent() {
      this.editStudentForm = true;
      this.$nextTick(() => {
        this.$refs.editForm.$el.scrollIntoView({ behavior: 'smooth' });
      });
    },
    addCourse() {
      this.selectedCourse = { code: '', name: '', credits: 0, grade: '' };
      this.editCourseForm = true;
      this.$nextTick(() => {
        this.$refs.editForm.$el.scrollIntoView({ behavior: 'smooth' });
      });
    },
    editCourse(course) {
      this.selectedCourse = { ...course };
      this.editCourseForm = true;
      this.$nextTick(() => {
        this.$refs.editForm.$el.scrollIntoView({ behavior: 'smooth' });
      });
    },
    deleteCourse(course) {
      axios
        .delete(`http://localhost:3001/students/1/courses/${course.code}`)
        .then(() => {
          this.student.courses = this.student.courses.filter(c => c.code !== course.code);
          this.fetchStudentData();
        })
        .catch(error => {
          console.error('Error deleting course:', error);
        });
    },
    saveStudent(updatedStudent) {
      axios.put('http://localhost:3001/students/1', updatedStudent).then(() => {
        this.fetchStudentData();
        this.editStudentForm = false;
      });
    },
    saveCourse(updatedCourse) {
      if (this.selectedCourse.code) {
        axios
          .put(
            `http://localhost:3001/students/1/courses/${this.selectedCourse.code}`,
            updatedCourse
          )
          .then(() => {
            this.fetchStudentData();
            this.editCourseForm = false;
          });
      } else {
        axios.post('http://localhost:3001/students/1/courses', updatedCourse).then(() => {
          this.fetchStudentData();
          this.editCourseForm = false;
        });
      }
    },
    cancelEdit() {
      this.editStudentForm = false;
      this.editCourseForm = false;
      this.selectedCourse = null;
    },
    startImageRotation() {
      setInterval(() => {
        this.currentImageIndex = (this.currentImageIndex + 1) % this.profileImages.length;
      }, 3000); // เปลี่ยนรูปทุก 3 วินาที
    },
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Prompt:wght@400;700&display=swap');

.app-container {
  position: relative;
  font-family: 'Prompt', sans-serif;
  padding: 30px;
  max-width: 1200px;
  width: 90%;
  margin: 30px auto;
  border-radius: 16px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.5);
  background-color: rgba(60, 50, 40, 0.7);
  color: #e0e0e0;
}

.background-video {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: -1;
  pointer-events: none;
}

.content {
  position: relative;
  z-index: 1;
  width: 100%;
}

.title {
  font-size: 2.5rem;
  color: #ffcc80;
  margin-bottom: 20px;
  text-align: center;
  font-weight: 600;
  background: linear-gradient(to right, rgba(255, 204, 128, 0.2), rgba(255, 179, 71, 0.2));
  padding: 15px 25px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
}

.student-container {
  background-color: rgba(70, 60, 50, 0.7);
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.profile-image-container {
  position: relative;
  width: 200px;
  height: 250px; /* เพิ่มความสูงเพื่อให้รูปภาพมีสัดส่วนที่ดีขึ้น */
}

.profile-img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover; /* ปรับให้รูปภาพเต็มกรอบโดยไม่เสียสัดส่วน */
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  transition: opacity 1s ease-in-out;
}

.student-info {
  text-align: center;
  width: 100%;
  background-color: rgba(80, 70, 60, 0.7);
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.2);
  margin-bottom: 20px;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  color: #e0e0e0;
}

.edit-button {
  background-color: rgba(255, 179, 71, 0.7);
}

.course-list {
  list-style-type: none;
  padding: 0;
  width: 100%;
}

.course-item {
  background-color: rgba(70, 60, 50, 0.7);
  padding: 15px;
  margin: 10px 0;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.add-course-button {
  background-color: rgba(129, 199, 132, 0.7);
}

.edit-course-button {
  background-color: rgba(255, 179, 71, 0.7);
}

.delete-course-button {
  background-color: rgba(229, 57, 53, 0.7);
}

h3 {
  color: #ffcc80;
  font-size: 1.8rem;
  margin-bottom: 15px;
}

h2 {
  color: #e0e0e0;
  font-size: 1.5rem;
  margin-bottom: 10px;
}

p {
  color: #e0e0e0;
  font-size: 1rem;
}

.button-column {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  gap: 5px;
}

button {
  box-sizing: border-box;
  text-align: center;
  font-family: inherit;
}

.divider {
  border: none;
  height: 1px;
  background-color: rgba(100, 90, 80, 0.7);
  margin: 15px 0;
  width: 100%;
}
</style>