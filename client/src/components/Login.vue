<template>
  <div class="login-container">
    <h1>User Login</h1>
    <form class="login-form" v-on:submit.prevent="onLogin">
      <div class="input-group">
        <label for="email">Email</label>
        <input type="email" id="email" name="email" v-model="email" required />
      </div>
      <div class="input-group">
        <label for="password">Password</label>
        <input
          type="password"
          id="password"
          name="password"
          v-model="password"
          required
        />
      </div>
      <button type="submit" class="login-button">Login</button>
      <div class="error" v-if="error">{{ error }}</div>
    </form>
  </div>
</template>
<script>
import AuthenService from "../services/AuthenService";
export default {
  data() {
    return {
      email: "",
      password: "",
      error: null,
    };
  },
  methods: {
    async onLogin() {
      try {
        const response = await AuthenService.login({
          email: this.email,
          password: this.password,
        });

        this.$store.dispatch("setToken", response.data.token);
        this.$store.dispatch("setUser", response.data.user);

        this.$router.push({
          name: "users",
        });
      } catch (error) {
        console.log(error);
        this.error = error.response.data.error;
        this.email = "";
        this.password = "";
      }
    },
  },
};
</script>
<style scoped>
.login-container {
  max-width: 400px;
  margin: 50px auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  text-align: center;
}
.login-container h1 {
  font-size: 24px;
  margin-bottom: 20px;
  color: #333;
}
.input-group {
  margin-bottom: 15px;
  text-align: left;
}
.input-group label {
  display: block;
  font-size: 14px;
  color: #555;
  margin-bottom: 5px;
}
.input-group input {
  width: 100%;
  padding: 10px;
  font-size: 14px;
  border: 1px solid #ccc;
  border-radius: 4px;
  transition: border-color 0.3s;
}
.input-group input:focus {
  border-color: #4caf50;
  outline: none;
}
.login-button {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  color: #fff;
  background-color: #4caf50;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}
.login-button:hover {
  background-color: #45a049;
}
.error {
  margin-top: 15px;
  color: red;
  font-size: 14px;
}
</style>
