<template>
    <!-- Welcome text in the top-left corner -->
    <div class="welcome-text">
      Step into my magical realm of movies!
    </div>
  
    <!-- Magician image below the text, floating up and down -->
    <img src="@/assets/magician.png" alt="Floating Magician" class="magician" />
  
    <div class="chatbot">
      <div class="chat-window" ref="chatWindow">
        <!-- Loop through messages and display them -->
        <div v-for="message in messages" :key="message.id" :class="['message', message.type]" >
          
        <span class="message-content" v-if="!message.isMovie">{{ message.content }}</span>
         <!-- Movie message with horizontal layout -->
         <div v-else class="movie-card">
            <div class="thumbnail-section">
            <img :src="message.content.thumbnail" alt="Movie Thumbnail" class="movie-thumbnail" />
            <div class="title-rating">
                <span class="movie-title">{{ message.content.title }}</span>
                <span class="movie-rating">Rating: {{ message.content.rating }}</span>
            </div>
            </div>

            <div class="movie-info">
            <p class="movie-synopsis">{{ message.content.synopsis }}</p>
            <p class="movie-runtime">Runtime: {{ message.content.runtime }} minutes</p>
            <p class="movie-cast">Cast: {{ message.content.cast.map(castMember => castMember.name).join(', ') }}</p>
            <p class="movie-crew">Crew: {{ message.content.crew.map(crewMember => crewMember.name).join(', ') }}</p>
            <a :href="message.content.watchLink" target="_blank" class="movie-link">Watch here</a>
            </div>
        </div>
        <div ref="lastMessage"></div>
    </div>
  </div>
  
      <!-- Option Selection (always visible) -->
      <div class="options" :class="{ 'with-input': optionSelected }">
        <button @click="selectOption(1)">Personalized Picks</button>
        <button @click="selectOption(2)">Discover by Genre</button>
      </div>
  
      <!-- Input Box (hidden initially) -->
      <div class="input-box" v-if="optionSelected">
        <input v-model="userInput" @keyup.enter="sendMessage" :placeholder="inputPlaceholder" />
        <button @click="sendMessage">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
            <path d="M2.5 10c0-.28.22-.5.5-.5h9.88l-3.8-3.8a.5.5 0 01.7-.7l5 5a.5.5 0 010 .7l-5 5a.5.5 0 01-.7-.7l3.8-3.8H3a.5.5 0 01-.5-.5z"/>
          </svg>
        </button>
      </div>
    </div>
  </template>
  
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        messages: [
          { id: Date.now(), type: 'bot', content: 'I’m the Film Wizard, ready to conjure up the best films for your next adventure. What cinematic spell shall I cast today?' }
        ],
        userInput: '',
        selectedOption: null,
        optionSelected: false,
        inputPlaceholder: ''
      };
    },
    methods: {
      selectOption(option) {
        this.optionSelected = true;
        this.userInput = '';
        if (option === 1) {
          this.inputPlaceholder = 'Tell me some movies you liked...';
          this.selectedOption = 1;
          this.messages.push({
            id: Date.now(),
            type: 'bot',
            content: 'Please tell me some movies you have watched and liked.'
          });
        } else if (option === 2) {
          this.selectedOption = 2;
          this.inputPlaceholder = 'Tell me a genre you want to watch...';
          this.messages.push({
            id: Date.now(),
            type: 'bot',
            content: 'Please tell me what genre you want to watch.'
          });
        }
  
        // Auto-scroll after a new option is selected
        this.$nextTick(() => this.scrollToBottom());
      },
      async sendMessage() {
        if (this.userInput.trim() === '') return;
  
        const userMessage = { id: Date.now(), type: 'user', content: this.userInput };
        this.messages.push(userMessage);
  
        let prompt;
        if (this.selectedOption === 1) {
          prompt = `I liked these movies: ${this.userInput}. Can you recommend some similar movies? Give me only the movie titles in a numbered list format.`;
        } else if (this.selectedOption === 2) {
          prompt = `I want to watch movies in the ${this.userInput} genre. Can you recommend some? Give me only the movie titles in a numbered list format.`;
        }
  
        try {
          const chatResponse = await axios.post('https://api.openai.com/v1/chat/completions', {
            model: 'gpt-3.5-turbo',
            messages: [{ role: 'user', content: prompt }]
          }, {
            headers: {
              'Authorization': `Bearer ${process.env.VUE_APP_OPENAI_API_KEY_1}`,
              'Content-Type': 'application/json'
            }
          });
  
          const recommendations = chatResponse.data.choices[0].message.content.split('\n').filter(line => line.trim());
          const cleanedRecommendations = recommendations.map(line => line.replace(/^\d+\.\s*/, '').trim());
  
          for (const movieTitle of cleanedRecommendations) {
            const movieDetails = await this.fetchMovieDetails(movieTitle.trim());
            if (movieDetails) {
              this.messages.push({
                id: Date.now() + Math.random(),
                type: 'bot',
                content: movieDetails,
                isMovie: true
              });
            }
          }
  
          if (this.messages.length > 0) {
            this.messages.push({
              id: Date.now() + 1,
              type: 'bot',
              content: `Here are some movies I found.`
            });
          } else {
            this.messages.push({
              id: Date.now() + 1,
              type: 'bot',
              content: 'Sorry, I couldn’t find any movies matching your request.'
            });
          }
        } catch (error) {
          console.error('Error communicating with ChatGPT or Movie API:', error);
          this.messages.push({ id: Date.now() + 2, type: 'bot', content: 'Sorry, something went wrong while fetching movie recommendations.' });
        }
  
        this.userInput = '';
  
        // Auto-scroll after the message is sent and response is received
        this.$nextTick(() => this.scrollToBottom());
      },
      async fetchMovieDetails(movieTitle) {
        try {
          const searchResponse = await axios.get('https://api.themoviedb.org/3/search/movie', {
            params: {
              api_key: process.env.VUE_APP_TMDB_API_KEY_2,
              query: movieTitle
            }
          });
  
          const movie = searchResponse.data.results[0];
          if (!movie) return null;
  
          const detailsResponse = await axios.get(`https://api.themoviedb.org/3/movie/${movie.id}`, {
            params: {
              api_key: process.env.VUE_APP_TMDB_API_KEY_2,
              append_to_response: 'videos,credits'
            }
          });
  
          const movieDetails = detailsResponse.data;
          return {
            id: movieDetails.id,
            title: movieDetails.title,
            thumbnail: `https://image.tmdb.org/t/p/w500${movieDetails.poster_path}`,
            synopsis: movieDetails.overview,
            rating: movieDetails.vote_average,
            runtime: movieDetails.runtime,
            cast: movieDetails.credits.cast.slice(0, 5),
            crew: movieDetails.credits.crew.slice(0, 5),
            watchLink: `https://www.themoviedb.org/movie/${movieDetails.id}`
          };
        } catch (error) {
          console.error(`Error fetching details for movie: ${movieTitle}`, error);
          return null;
        }
      },
      scrollToBottom() {
        const chatWindow = this.$refs.chatWindow;
         if (chatWindow) {
             chatWindow.scrollTo({
             top: chatWindow.scrollHeight,
             behavior: 'smooth'  // Enables smooth scrolling
    });
  }
}


    }
  };
  </script>

<style>
html, body {
  background-color: rgb(0, 0, 0);
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
}

.welcome-text {
  position: absolute;
  top: 4vw;
  left: 4vw;
  font-size: 5.5vw;
  color: #d30024;
  font-family: "Handjet", sans-serif;
  overflow: hidden;
  max-width: calc(100vw - 50vw - 10vw);
  white-space: normal;
  word-wrap: normal;
}

@keyframes typing {
  from { width: 0; }
  to { width: 100%; }
}

.magician {
  position: absolute;
  top: 22vw;
  left: 5vw;
  bottom: 8vw;
  height: 50vh;
  width: 35vw;
  animation: float 2.5s ease-in-out infinite;
}

@keyframes float {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-30px);
  }
}

.chatbot {
  display: flex;
  flex-direction: column;
  position: absolute;
  right: 5%;
  height: 90vh;
  width: 50%;
  max-width: 80vw;
  border: 1px solid #141414;
  border-radius: 20px;
  overflow: hidden;
  background-color: #141414;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.8);
  margin: 0 auto;
}

.chat-window {
  flex: 1;
  padding: 2%;
  overflow-y: scroll;
  background-color: #141414;
}

.message {
  margin-bottom: 1.5%;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.message.user {
  align-items: flex-end;
}

.message-content {
  max-width: 80%;
  padding: 1.5%;
  border-radius: 20px;
  font-size: 1.5vw;
  line-height: 1.6;
  color: #FFFFFF;
}

.message.user .message-content {
  background-color: #ff042f;
  border-bottom-right-radius: 0;
}

.message.bot .message-content {
  background-color: #221F1F;
  font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
  color: #FFFFFF;
  border-bottom-left-radius: 0;
}

.options {
  display: flex;
  justify-content: space-around;
  padding: 2%;
  transition: transform 0.3s ease-in-out;
}

.options.with-input {
  margin: 0%;
  padding: 0%;
  background-blend-mode: light;
}

.input-box {
  display: flex;
  align-items: center;
  padding-top: 0%;
  padding-left: 1vw;
  padding-right: 1vw;
  padding-bottom: 1vh;
  background-color: #141414;
}

.options button {
  background-color: #ff042f;
  color: #FFFFFF;
  border: none;
  font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
  font-size: 2vw;
  margin: 2%;
  flex: 1;
  border-radius: 10px;
  padding: 1%;
  cursor: pointer;
}

input {
  flex: 1;
  padding: 1.5%;
  border: 1px solid #B3B3B3;
  border-radius: 20px;
  margin-right: 2%;
  font-size: 2vw;
  background-color: #221F1F;
  color: #FFFFFF;
}

input::placeholder {
  color: #E6E6E6;
}

button {
  background-color: transparent;
  border: none;
  color: #ff042f;
  padding: 0;
  outline: none;
  cursor: pointer;
}

button svg {
  width: 5vw;
  height: 5vw;
}

/* Movie Card styling for the new layout */
.movie-card {
  display: flex;
  flex-direction: row;
  padding: 1%;
  background-color: #1a1a1a;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  align-items: flex-start;
  margin-bottom: 20px; /* Space between cards */
}

/* Thumbnail and Title Section */
.thumbnail-section {
  width: 50%;
  max-width: 500px;
  margin-right: 2%; /* Space between thumbnail and movie info */
  display: flex;
  flex-direction: column;
}

.movie-thumbnail {
  width: 100%;
  border-radius: 10px;
  margin-top: 12px;
  margin-bottom: 10px; /* Space below the thumbnail */
}

/* Title and rating below the thumbnail, each on a new line */
.title-rating {
  display: flex;
  flex-direction: column;
  margin-top: 10px;
  color: #FFFFFF;
}

.movie-title {
  font-size: 1.8vw;
  font-weight: bold;
  color: #FFFFFF;
  margin-bottom: 5px;
}

.movie-rating {
  font-size: 1.4vw;
  color: #f39c12;
}

/* Movie Info on the right side */
.movie-info {
  flex: 1;
  font-size: 1.5vw;
  color: #E6E6E6;
}

.movie-synopsis,
.movie-runtime,
.movie-cast,
.movie-crew {
  margin-top: 10px;
  color: #B3B3B3;
}

.movie-synopsis {
  font-size: 0.4vw;
}

.movie-link {
  margin-top: 10px;
  font-size: 1.5vw;
  color: #ff042f;
  text-decoration: none;
}

/* Responsive behavior */
@media (max-width: 768px) {
  .movie-card {
    flex-direction: column;
  }

  .movie-thumbnail {
    width: 100%;
  }

  .movie-info {
    margin-top: 20px;
  }
}

/* Adjustments for large screens */
@media (min-width: 1200px) {
  .movie-title {
    font-size: 2.2vw;
  }

  .movie-rating {
    font-size: 1.6vw;
  }

  .movie-synopsis {
    font-size: 1.8vw;
  }
}
</style>



 
 
 
 
 
 
 
 
 
 