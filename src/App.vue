<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'

// Define reactive state variables
const firstname = ref<string>('')
const lastname = ref<string>('')
const username = ref<string>('')
const password = ref<string>('')
const role = ref<string>('')
const userId = ref<number | null>(null)
const avatarId = ref<number | null>(null)
const selectedFile = ref<File | null>(null)
const token = ref<string>('')
const responseAPI = ref<any[]>([])

const register = () => {
  const formData = {
    firstname: firstname.value,
    lastname: lastname.value,
    username: username.value,
    password: password.value,
    role: role.value
  }

  axios
    .post('http://localhost:8080/register', formData)
    .then((response) => {
      console.log('Register Response:', response.data)
      token.value = response.data.token
    })
    .catch((error) => {
      console.error('Register Error:', error)
    })
}

const login = () => {
  const formData = {
    username: username.value,
    password: password.value
  }

  axios
    .post('http://localhost:8080/login', formData)
    .then((response) => {
      console.log('Login Response:', response.data)
      token.value = response.data.token
    })
    .catch((error) => {
      console.error('Login Error:', error)
    })
}

const getRandomFloat = (min: number, max: number): number => {
  return Math.random() * (max - min) + min
}

const getRandomDate = (): string => {
  const start = new Date(2024, 0, 1).getTime() // January 1, 2024
  const end = new Date().getTime() // Current date
  const randomTime = start + Math.random() * (end - start)
  const randomDate = new Date(randomTime)
  return randomDate.toISOString()
}

const addRace = () => {
  const data = {
    averageSpeed: getRandomFloat(50, 150), // Random speed between 50 and 150
    distanceCovered: getRandomFloat(100, 200), // Random distance between 100 and 200
    timeSpent: getRandomFloat(1, 3), // Random time between 1 and 3 hours
    wheelRotationSpeed: getRandomFloat(80, 120), // Random speed between 80 and 120
    createdAt: getRandomDate(), // Random date
    user: {
      id: userId.value // Random user ID between 1 and 100
    }
  }

  axios
    .post('http://localhost:8080/races', data, {
      headers: {
        Authorization: `Bearer ${token.value}`,
        'Content-Type': 'application/json'
      }
    })
    .then((response) => {
      console.log('Response:', response.data)
      responseAPI.value = response.data
    })
    .catch((error) => {
      console.error('Error:', error)
    })
}

// Get races function
const getRacesById = () => {
  if (userId.value === null) {
    console.error('User ID is required')
    return
  }

  axios
    .get(`http://localhost:8080/users/${userId.value}/races`, {
      headers: {
        Authorization: `Bearer ${token.value}`
      }
    })
    .then((response) => {
      console.log('Races Response:', response.data)
      responseAPI.value = response.data
    })
    .catch((error) => {
      console.error('Get Races Error:', error)
    })
}

const getAllUsers = () => {
  if (userId.value === null) {
    console.error('User ID is required')
    return
  }

  axios
    .get(`http://localhost:8080/users`, {
      headers: {
        Authorization: `Bearer ${token.value}`
      }
    })
    .then((response) => {
      console.log('Races Response:', response.data)
      responseAPI.value = response.data
    })
    .catch((error) => {
      console.error('Get Races Error:', error)
    })
}

const getAllRaces = () => {
  if (userId.value === null) {
    console.error('User ID is required')
    return
  }

  axios
    .get(`http://localhost:8080/races`, {
      headers: {
        Authorization: `Bearer ${token.value}`
      }
    })
    .then((response) => {
      console.log('Races Response:', response.data)
      responseAPI.value = response.data
    })
    .catch((error) => {
      console.error('Get Races Error:', error)
    })
}

const getUsersByUsername = () => {
  axios
    .get(`http://localhost:8080/users`, {
      headers: {
        Authorization: `Bearer ${token.value}`
      }
    })
    .then((response) => {
      console.log('Races Response:', response.data)
      responseAPI.value = response.data.filter((user: any) => (user.username = username.value))
    })
    .catch((error) => {
      console.error('Get Races Error:', error)
    })
}

const handleFileChange = (event: Event) => {
  const file = (event.target as HTMLInputElement).files?.[0]
  if (file) {
    selectedFile.value = file
  }
}

const uploadAvatar = () => {
  const formData = new FormData()
  formData.append('file', selectedFile.value)

  axios
    .post('http://localhost:8080/documents', formData, {
      headers: {
        Authorization: `Bearer ${token.value}`,
        'Content-Type': 'multipart/form-data'
      }
    })
    .then((response) => {
      console.log('Image Response:', response.data)
      avatarId.value = response.data.id
    })
    .catch((error) => {
      console.error('Get Races Error:', error)
    })
}

const updateAvatar = () => {
  axios
    .patch(
      `http://localhost:8080/users/${userId.value}/avatar/${avatarId.value}`,
      {},
      {
        headers: {
          Authorization: `Bearer ${token.value}`
        }
      }
    )
    .then((response) => {
      console.log('Av Response:', response.data)
      responseAPI.value = response.data
    })
    .catch((error) => {
      console.error('Get img up Error:', error)
    })
}

const displayAvatar = () => {
  // Fetch the image Blob from an API
  axios
    .get(`http://localhost:8080/documents/${avatarId.value}`, {
      headers: {
        Authorization: `Bearer ${token.value}`
      },
      responseType: 'blob' // Set the response type to 'blob'
    })
    .then((response) => {
      console.log('Img Response:', response)

      const blob = response.data // The response is the blob

      // Create a local URL for the blob
      const imgUrl = URL.createObjectURL(blob)

      // Set the img element's src attribute to the blob URL
      const imgElement = document.getElementById('image') as HTMLImageElement
      if (imgElement) {
        imgElement.src = imgUrl
      }
    })
    .catch((error) => {
      console.error('Error fetching the image:', error)
    })
}
</script>

<template>
  <div>
    <h1>Register</h1>
    <input v-model="firstname" type="text" placeholder="Prénom" /><br />
    <input v-model="lastname" type="text" placeholder="Nom" /><br />
    <input v-model="username" type="text" placeholder="Nom d'utilisateur" /><br />
    <input v-model="password" type="password" placeholder="Mot de passe" /><br />
    <input v-model="role" type="text" placeholder="Role" /><br />
    <button @click="register">Register</button>
    <div>
      <h2>Response : token</h2>
      {{ token }}
    </div>

    <h1>Login</h1>
    <input v-model="username" type="text" placeholder="Nom d'utilisateur" /><br />
    <input v-model="password" type="password" placeholder="Mot de passe" /><br />
    <button @click="login">Login</button>
    <div>
      <h2>Response : token</h2>
      {{ token }}
    </div>

    <h1>Add race to a User</h1>
    <input v-model.number="userId" type="number" placeholder="User ID" /><br />
    <button @click="addRace">Add Race</button>
    <div>
      <h2>Response</h2>
      <pre>{{ responseAPI }}</pre>
      <!-- Use <pre> to format JSON response -->
    </div>

    <h1>Get Races by User ID</h1>
    <input v-model.number="userId" type="number" placeholder="User ID" /><br />
    <button @click="getRacesById">Get Races</button>
    <div>
      <h2>Response</h2>
      <pre>{{ responseAPI }}</pre>
      <!-- Use <pre> to format JSON response -->
    </div>
  </div>

  <h1>Get all Users</h1>
  <button @click="getAllUsers">Get all users</button>
  <div>
    <h2>Response</h2>
    <pre>{{ responseAPI }}</pre>
    <!-- Use <pre> to format JSON response -->
  </div>

  <h1>Get user by username</h1>
  <input v-model="username" type="text" placeholder="username" /><br />
  <button @click="getUsersByUsername">Get user</button>
  <div>
    <h2>Response</h2>
    <pre>{{ responseAPI }}</pre>
    <!-- Use <pre> to format JSON response -->
  </div>

  <h1>Get all races</h1>
  <button @click="getAllRaces">Get all users</button>
  <div>
    <h2>Response</h2>
    <pre>{{ responseAPI }}</pre>
    <!-- Use <pre> to format JSON response -->
  </div>

  <h1>Choose img</h1>
  <input type="file" @change="handleFileChange" /><br />
  <button @click="uploadAvatar">Upload</button><br />

  <h1>Update avatar</h1>
  <button @click="updateAvatar">Upload</button><br />

  <h1>Display avatar</h1>
  <img id="image" alt="Blob Image" /><br />
  <button @click="displayAvatar()">img</button>
</template>

<style scoped>
/* Add your scoped CSS here */
</style>
