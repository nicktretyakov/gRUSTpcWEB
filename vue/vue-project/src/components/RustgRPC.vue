<template>
    <div>
      <h1>gRPC Web Client</h1>
      <input v-model="name" placeholder="Enter your name" />
      <button @click="sayHello">Say Hello</button>
      <p v-if="response">{{ response }}</p>
    </div>
  </template>
  
  <script>
  import { HelloRequest } from './greeter_pb';  // Generated protobufs
  import { GreeterClient } from './GreeterServiceClientPb';  // Generated service client
  
  export default {
    data() {
      return {
        name: '',
        response: '',
        client: null,
      };
    },
    mounted() {
      this.client = new GreeterClient('http://localhost:8080');  // gRPC-Web proxy URL
    },
    methods: {
      sayHello() {
        const request = new HelloRequest();
        request.setName(this.name);
  
        this.client.sayHello(request, {}, (err, response) => {
          if (err) {
            console.error('Error:', err.message);
            return;
          }
          this.response = response.getMessage();
        });
      },
    },
  };
  </script>
  
  <style scoped>
  input {
    margin-right: 10px;
  }
  </style>