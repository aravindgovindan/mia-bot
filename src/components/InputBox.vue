<template>
  <div>
    <input type="textarea" v-model="query" @keyup.enter="analyzeQuery" />
    <div v-if="action || type || filter">
      <p>Action: {{ action }}</p>
      <p>Type: {{ type }}</p>
      <p>Filter: {{ filter }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      query: "",
      action: "",
      type: "",
      filter: "",
    };
  },
  methods: {
    async analyzeQuery() {
      const prompts = [
        { role: 'user', content: 'You are a Helpbot for an internal application. Users will ask you questions to help them with something. You should breakdown their query and respond with a json object with keys: action, type, filter.' },
        { role: 'assistant', content: 'Okay. I will do that.' },
        { role: 'user', content: 'I want to create a new content record' },
        { role: 'assistant', content: '{"action": "create", "type": "content", "filter": {}}' },
        { role: 'user', content: 'I want to change the metadata of particle with id X18273' },
        { role: 'assistant', content: '{"action": "edit",  "type": "particle",  "filter": {"id": "X18273"}}' },
        { role: 'user', content: this.query }
      ]
      const response = await fetch("https://api.openai.com/v1/chat/completions", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${process.env.VUE_APP_OPENAI_API_KEY}`,
        },
        body: JSON.stringify({
          model: 'gpt-3.5-turbo',
          messages: prompts,
        }),
      });
      const data = await response.json();
      const choices = data?.choices[0];
      const content = JSON.parse(choices.message.content)
      console.log(content)
      const { action, type, filter } = content;
      this.action = action;
      this.type = type;
      this.filter = filter;
    },
  },
};
</script>
