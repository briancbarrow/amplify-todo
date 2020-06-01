<template>
  <amplify-authenticator>
    <div id="app">
      <h1>Todo App</h1>
      <input type="text" v-model="name" placeholder="Todo name" />
      <input type="text" v-model="description" placeholder="Todo description" />
      <button v-on:click="createTodo">Create Todo</button>
      <div v-for="item in todos" :key="item.id">
        <h3>{{ item.name }}</h3>
        <p>{{ item.description }}</p>
      </div>
    </div>
    <amplify-sign-out></amplify-sign-out>
  </amplify-authenticator>
</template>

<script>
import API, { graphqlOperation } from "@aws-amplify/api";
import { Auth } from "@aws-amplify/auth";
import { createTodo } from "./graphql/mutations";
import { listTodos } from "./graphql/queries";
import { onCreateTodo } from "./graphql/subscriptions";

export default {
  name: "app",
  data() {
    return {
      name: "",
      description: "",
      todos: [],
      user: {}
    };
  },
  // created() {
  //   this.getUserInfo();
  // },
  mounted() {
    // console.log("this user id", this.user.id);
    this.getTodos();
    this.subscribe();
  },
  methods: {
    async createTodo() {
      const {
        name,
        description,
        user: { id: userId }
      } = this;
      if (!name || !description) return;
      console.log("DID i get it", userId);
      const todo = { name, description, userId };
      await API.graphql({
        query: createTodo,
        variables: { input: todo }
      });
      this.name = "";
      this.description = "";
    },
    async getTodos() {
      await this.getUserInfo();
      const filter = { userId: { eq: this.user.id } };
      console.log("GET TODOS USER", filter);
      const todos = await API.graphql(
        graphqlOperation(listTodos, { filter: filter })
      );
      console.log("todos", todos);
      this.todos = todos.data.listTodos.items;
    },
    async getUserInfo() {
      console.log("USER Enter");
      const user = await Auth.currentUserInfo();
      console.log("USER After", user);
      this.user = user;
      return user;
    },
    subscribe() {
      API.graphql({ query: onCreateTodo }).subscribe({
        next: eventData => {
          console.log("EVENT DATA", eventData);
          let todo = eventData.value.data.onCreateTodo;
          if (this.todos.some(item => item.name === todo.name)) return;
          this.todos = [...this.todos, todo];
        }
      });
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
