<template>
  <div v-if="active && isStaff" class="actor-container">
    <div class="add-actor" v-if="createMode">
      <div class="header">Add Actor</div>
      <div class="note">Please fill in the fields in details</div>
      <div class="input-container">
        <input-item label="First name" width="wide" v-model="firstName"></input-item>
        <input-item label="Last name" width="wide" v-model="lastName"></input-item>
      </div>
      <action-button label="Submit" @click="addActor"></action-button>
    </div>
    <div class="actors">
      <div class="header">All Actors</div>
      <div class="container">
        <div
          v-for="(actor, index) in actors"
          :key="actor.id"
          :class="index % 2 === 0 ? 'odd row' : 'row'"
        >
          <span>{{actor.first_name}} {{actor.last_name}}</span>
          <button class="delete-btn" @click="deleteActor(actor.id)">✕</button>
        </div>
      </div>
      <add-btn @click="createMode = !createMode"></add-btn>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

import AddBtn from '../comps/AddBtn.vue';
import InputItem from '../comps/InputItem.vue';
import ActionButton from '../comps/ActionButton.vue';
export default {
  props: {
    isStaff: {
      type: Boolean,
      default: false
    }
  },
  data: () => ({
    active: false,
    actors: [],
    createMode: false,
    firstName: '',
    lastName: ''
  }),
  computed: {
    token () {
      return localStorage.getItem('access');
    }
  },
  methods: {
    async fetchActors () {
      try {
        const { data: actors } = await axios.get(`${import.meta.env.VITE_API_URL}/api/cinema/actors/`, {
          headers: { Authorization: `Bearer ${this.token}` }
        });
        this.actors = actors;
      } catch (err) {
        console.error(err.response.data);
      }
    },

    async addActor () {
      try {
        const config = {
          headers: {
            Authorization: `Bearer ${this.token}`,
            'Content-Type': 'application/json'
          }
        };

        await axios.post(
          `${import.meta.env.VITE_API_URL}/api/cinema/actors/`,
          {
            first_name: this.firstName,
            last_name: this.lastName
          },
          config
        );

        this.createMode = !this.createMode;
        this.fetchActors();

        this.firstName = '';
        this.lastName = '';
      } catch (err) {
        console.error(err);
      }
    },

    async deleteActor (id) {
      if (!confirm('Delete this actor?')) return;
      try {
        await axios.delete(
          `${import.meta.env.VITE_API_URL}/api/cinema/actors/${id}/`,
          {
            headers: { Authorization: `Bearer ${this.token}` }
          }
        );
        this.actors = this.actors.filter((actor) => actor.id !== id);
      } catch (err) {
        console.error(err.response?.data || err);
        alert('Failed to delete actor.');
      }
    },

    hashHandler () {
      this.active = Boolean(location.hash.match('actors$'));
    }
  },
  watch: {
    active () {
      if (this.active) {
        this.fetchActors();
      }
    }
  },
  mounted () {
    window.addEventListener('hashchange', this.hashHandler);
    this.hashHandler();
  },
  beforeDestroy () {
    window.removeEventListener('hashchange', this.hashHandler);
  },

  components: {
    AddBtn,
    InputItem,
    ActionButton
  }

};
</script>

<style scoped>
.actor-container, .actor-container > * {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
  width: 100%;
}

.add-actor {
  margin-bottom: 36px;
}

.action-button {
  margin-top: 36px;
}

.header {
  font-weight: 600;
  font-size: 50px;
  line-height: 61px;
}

.note {
  font-size: 25px;
  line-height: 31px;
}

.container {
  width: 100%;
  font-size: 25px;
  line-height: 30px;
}

.container > .row {
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 10px 0 16px;
  border-radius: 10px;
}

.container > .odd {
  background-color: var(--secondary-bg);
}

.delete-btn {
  background: transparent;
  border: none;
  color: var(--main-font);
  cursor: pointer;
  font-size: 18px;
  padding: 4px 8px;
}
.delete-btn:hover {
  color: var(--red);
}

.input-container {
  width: 100%;
  display: flex;
  gap: 100px;
}
</style>
