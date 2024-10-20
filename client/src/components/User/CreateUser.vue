<template>
  <div class="register-container">
    <h1>Register</h1>
    <form @submit.prevent="createUser" class="register-form">
      <div class="form-group">
        <label for="name">Name:</label><br>
        <input type="text" id="name" v-model="user.name" required />
      </div>
      <div class="form-group">
        <label for="lastname">Last Name:</label><br>
        <input type="text" id="lastname" v-model="user.lastname" required />
      </div>
      <div class="form-group">
        <label for="email">Email:</label><br>
        <input type="email" id="email" v-model="user.email" required />
      </div>
      <div class="form-group">
        <label for="password">Password:</label><br>
        <input type="password" id="password" v-model="user.password" required />
      </div>
      <div class="form-group button-container">
        <button type="submit" class="submit-button">Create User</button>
      </div>
    </form>
  </div>
</template>

<script>
import UsersService from '../../services/UsersService';

export default {
  data() {    
    return {
      users: [],
      user: {}
    };
  },
  async created() {
    try {
      this.users = (await UsersService.index()).data;
    } catch (err) {
      console.log(err);
    }
  },
  methods: {
    async createUser() {
      try {
        await UsersService.post(this.user);
        this.$router.push('/users');
      } catch (err) {
        console.log(err);
      }
    },
    navigateTo(route) {
      this.$router.push(route);
    }
  }
};
</script>
<style scoped>
.register-container {
  max-width: 400px;
  margin: 0 auto;
  padding: 40px;
  border: 1px solid #000000;
  border-radius: 8px;
  box-shadow: 0 0 10px rgb(238, 236, 236);
  background-color: #cccc;
  color: #cccc;
}

h1 {
  text-align: center;
  color: #000000;
  font-size: 24px;
}

.register-form {
  display: flex;
  flex-direction: column;
}

.form-group {
  margin-bottom: 15px;
}

label {
  font-weight: bold;
  color: #000000;
  font-size: 16px;
}

input {
  padding: 10px;
  border: 1px solid #000000;
  border-radius: 4px;
  outline: none;
  transition: border-color 0.3s;
  color: #000000;
  background-color: #ffffff;
  font-size: 16px;
}

input:focus {
  border-color: #ffcc00;
}

.submit-button {
  background-color: #ffcc00;
  color: #333;
  padding: 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 18px;
  transition: background-color 0.3s;
}

.submit-button:hover {
  background-color: #e6b800;
}
</style>
