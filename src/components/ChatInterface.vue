<template>
    <div class="chat-container">
      <!-- Display area for chat history -->
      <div class="chat-history">
        <div v-for="(entry, index) in chatHistory" :key="index" :class="entry.sender">
          <strong>{{ entry.sender }}:</strong> {{ entry.message }}
        </div>
      </div>
  
      <!-- Input field for user messages and Send button -->
      <div class="chat-input">
        <input
          v-model="messageInput"
          type="text"
          placeholder="Type your message..."
          @keyup.enter="sendMessage"
        />
        <button @click="sendMessage">Send</button>
      </div>
    </div>
  </template>
  
  <script>
  import { ref } from 'vue';
  
  export default {
    setup() {
      const messageInput = ref('');
      const chatHistory = ref([]);
  
      const sendMessage = async () => {
        if (messageInput.value.trim() !== '') {
          // Add user message to chat history
          chatHistory.value.push({
            sender: 'User',
            message: messageInput.value,
          });
  
          const userMessage = messageInput.value;
          messageInput.value = '';
  
          try {
            const response = await fetch('/api/chat', {
              method: 'POST',
              headers: {
                'Content-Type': 'application/json',
              },
              body: JSON.stringify({ message: userMessage }),
            });
  
            const data = await response.json();
            console.log('Response from server:', data);
  
            // Updated to match the actual API response structure
            if (data.reply && data.reply.content) {
              const aiMessage = data.reply.content[0].text;
              chatHistory.value.push({
                sender: 'AI',
                message: aiMessage,
              });
            } else {
              chatHistory.value.push({
                sender: 'Error',
                message: 'No valid reply from AI.',
              });
            }
          } catch (error) {
            console.error('Error fetching response:', error);
            chatHistory.value.push({
              sender: 'Error',
              message: 'Failed to get response from AI.',
            });
          }
        }
      };
  
      return {
        messageInput,
        chatHistory,
        sendMessage,
      };
    },
  };
  </script>
  
  <style scoped>
  .chat-container {
    max-width: 600px;
    margin: 0 auto;
    border: 1px solid #ccc;
    border-radius: 8px;
    padding: 16px;
    display: flex;
    flex-direction: column;
  }
  
  .chat-history {
    max-height: 300px;
    overflow-y: auto;
    margin-bottom: 10px;
    padding: 10px;
    background-color: #f9f9f9;
    border-radius: 4px;
  }
  
  .User {
    text-align: right;
    color: blue;
  }
  
  .AI {
    text-align: left;
    color: green;
  }
  
  .chat-input {
    display: flex;
  }
  
  input[type="text"] {
    flex: 1;
    padding: 8px;
    font-size: 16px;
    border-radius: 4px;
    border: 1px solid #ddd;
  }
  
  button {
    padding: 8px 16px;
    font-size: 16px;
    background-color: #bb0b2b;
    color: #fff;
    border: none;
    border-radius: 4px;
    margin-left: 5px;
    cursor: pointer;
  }
  </style>