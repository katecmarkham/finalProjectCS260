<template>
<div class="admin">
    <h1>Add, edit, or delete a hike!</h1>
    <div class="heading">
      <div class="circle">1</div>
      <h2>Add a hike!</h2>
    </div>
    <div class="add">
      <div class="form">
        <input v-model="title" placeholder="Title">
        <p></p>
        <input v-model="location" placeholder="Location">
        <p></p>
        <input v-model="difficulty" placeholder="Difficulty">
        <p></p>
        <input v-model="description" placeholder="Description">
        <p></p>
        <input type="file" name="photo" @change="fileChanged">
        <button @click="upload">Upload</button>
      </div>
      <div class="upload" v-if="addHike">
        <h2>{{addHike.title}}</h2>
        <h4>{{addHike.location}}</h4>
        <h4>{{addHike.difficulty}}</h4>
        <p> {{addHike.description}}</p>
        <img :src="addHike.path" />
      </div>
    </div>
        <div class="heading">
      <div class="circle">2</div>
      <h2>Edit/Delete a hike!</h2>
    </div>
    <div class="edit">
      <div class="form">
        <input v-model="findTitle" placeholder="Search">
        <div class="suggestions" v-if="suggestions.length > 0">
          <div class="suggestion" v-for="s in suggestions" :key="s.id" @click="selectItem(s)">{{s.title}}
          </div>
        </div>
      </div>
      <div class="upload" v-if="findItem">
        <input v-model="findItem.title">
        <p></p>
        <input v-model="findItem.location">
        <p></p>
        <input v-model="findItem.difficulty">
        <p></p>
        <input v-model="findItem.description">
        <p></p>
        <img :src="findItem.path" />
      </div>
      <div class="actions" v-if="findItem">
        <button @click="deleteItem(findItem)">Delete</button>
        <button @click="editItem(findItem)">Edit</button>
      </div>
    </div>
</div>

</template>


<script>
import axios from 'axios';
export default {
  name: 'Admin',
  data() {
    return {
      hikes: [],
      findTitle: "",
      findItem: null,
      title: "",
      location: "",
      difficulty: "",
      description: "",
      file: null,
      addHike: null,
    }
  },
  computed: {
    suggestions() {
      let hikes = this.hikes.filter(hike => hike.title.toLowerCase().startsWith(this.findTitle.toLowerCase()));
      return hikes.sort((a, b) => a.title > b.title);
    }
  },
  created() {
    this.getHikes();
  },
  methods: {
    async upload() {
      try {
        const formData = new FormData();
        formData.append('photo', this.file, this.file.name)
        let r1 = await axios.post('/api/photos', formData);
        console.log(this.description);
        let r2 = await axios.post('/api/hikes', {
          title: this.title,
          location: this.location,
          difficulty: this.difficulty,
          description: this.description,
          path: r1.data.path
        });
        console.log(r2.data);
        this.addHike = r2.data;
      } catch (error) {
        console.log(error);
      }
    },
    fileChanged(event) {
      this.file = event.target.files[0]
    },
    async getHikes() {
      try {
        let response = await axios.get("/api/hikes");
        this.hikes = response.data;
        return true;
      } catch (error) {
        console.log(error);
      }
    },
    selectItem(item) {
      this.findTitle = "";
      this.findItem = item;
    },
    async deleteItem(item) {
      try {
        await axios.delete("/api/hikes/" + item._id);
        this.findItem = null;
        this.getHikes();
        return true;
      } catch (error) {
        console.log(error);
      }
    },
    async editItem(item) {
      try {
        await axios.put("/api/hikes/" + item._id, {
          title: this.findItem.title,
          location: this.findItem.location,
          difficulty: this.findItem.difficulty,
          description: this.findItem.description,
        });
        this.findItem = null;
        this.getHikes();
        return true;
      } catch (error) {
        console.log(error);
      }
    },
  }
}
</script>


<style scoped>
.image h2 {
  font-style: italic;
  font-size: 1em;
}

.heading {
  display: flex;
  margin-bottom: 20px;
  margin-top: 20px;
}

.heading h2 {
  margin-top: 8px;
  margin-left: 10px;
}

.add,
.edit {
  display: flex;
}

.circle {
  border-radius: 50%;
  width: 18px;
  height: 18px;
  padding: 8px;
  background: #333;
  color: #fff;
  text-align: center
}

/* Form */
input,
textarea,
select,
button {
  font-family: 'Montserrat', sans-serif;
  font-size: 1em;
}

.form {
  margin-right: 50px;
}

/* Uploaded images */
.upload h2 {
  margin: 0px;
}

.upload img {
  max-width: 300px;
}

/* Suggestions */
.suggestions {
  width: 200px;
  border: 1px solid #ccc;
}

.suggestion {
  min-height: 20px;
}

.suggestion:hover {
  background-color: #5BDEFF;
  color: #fff;
}
</style>
