<template>
  <div>
    <h1>Edit User</h1>
    <form v-on:submit.prevent="editUser">
      <div class="form-group">
        <label for="name">Name:</label>
        <input type="text" v-model="user.name" id="name" />
      </div>

      <div class="form-group">
        <label for="lastname">Last Name:</label>
        <input type="text" v-model="user.lastname" id="lastname" />
      </div>

      <div class="form-group">
        <label for="email">Email:</label>
        <input type="text" v-model="user.email" id="email" />
      </div>

      <div class="form-group">
        <label for="password">Password:</label>
        <input type="text" v-model="user.password" id="password" />
      </div>

      <div class="form-buttons">
        <button type="submit">Edit User</button>
      </div>
    </form>
  </div>
</template>

<script>
import UsersService from "@/services/UsersService";

export default {
  data() {
    return {
      user: {
        name: "",
        lastname: "",
        email: "",
        password: "",
        status: "active",
      },
    };
  },
  async created() {
    try {
      const userId = this.$route.params.userId;
      const response = await UsersService.show(userId);
      this.user = response.data;
    } catch (err) {
      console.error(err);
    }
  },
  methods: {
    async editUser() {
      try {
        await UsersService.put(this.user);
        this.$router.push("/users");
      } catch (err) {
        console.error(err);
      }
    },
  },
};
</script>

<style scoped>
div {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

form {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  max-width: 600px;
  width: 100%;
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input[type="text"] {
  width: 100%;
  padding: 10px;
  margin: 5px 0;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 1em;
}

.form-buttons {
  display: flex;
  justify-content: center;
}

button {
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  font-size: 1em;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #45a049;
}

@media (max-width: 768px) {
  form {
    padding: 15px;
  }

  button {
    width: 100%;
  }
}
</style>
