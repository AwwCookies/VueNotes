<template>
  <main>
    <div class="overlay" v-show="showModal">
      <div class="modal" ref="modalRef">
        <div class="error" v-show="error">Error: {{ error }}</div>
        <textarea v-model.trim="newNote" name="notes" id="notes" rows="20" ref="textAreaRef"></textarea>
        <button @click="addNote">Add Note</button>
        <button class="close" @click="toggleModal(false)">Close</button>
      </div>
    </div>
    <div class="container">
      <header>
        <h1>Notes</h1>
        <button @click="toggleModal(true)">+</button>
      </header>
      <div class="cards-container">
        <div @click.right.stop="deleteNote($event, note.id)" class="card" v-for="note in notes"
          :style="{ backgroundColor: note.backgroundColor }">
          <p class="main-text">{{ note.text }}</p>
          <div class="date">{{ note.date }}</div>
        </div>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, nextTick } from "vue";
import { useLocalStorage, onClickOutside, useEventListener } from "@vueuse/core";

const showModal = ref(false);
const textAreaRef = ref(null);
const modalRef = ref(null);
const newNote = ref("");
const notes = useLocalStorage('notes', [])
const error = ref("");

const noteMinLength = 10;
const noteMaxLength = 200;

// close modal when clicking outside of it
onClickOutside(modalRef, () => {
  if (showModal.value) toggleModal(false);
});

// close modal when pressing escape key
useEventListener(modalRef, 'keydown', (e) => {
  if (e.key === 'Escape') toggleModal(false);
})

// add note when pressing enter in textarea
useEventListener(textAreaRef, 'keydown', (e) => {
  if (e.key === 'Enter') addNote();
})

// generate random color
function getRandomColor() {
  // https://stackoverflow.com/questions/1484506/random-color-generator
  return `hsl(${Math.floor(Math.random() * (360 - 1 + 1) + 1)}, 50%, 50%)`;
}

const addNote = () => {
  // validate
  if (!newNote.value) return error.value = "Please enter a note";
  if (newNote.value.length > noteMaxLength) {
    return error.value = `Note is too long ${newNote.value.length}/${noteMaxLength} chars`;
  }
  if (newNote.value.length < noteMinLength) {
    return error.value = `Note is too short ${newNote.value.length}/${noteMinLength} chars`;
  }

  const note = {
    id: Math.floor(Math.random() * (100000 - 1 + 1) + 1),
    text: newNote.value,
    date: new Date(),
    backgroundColor: getRandomColor()
  }

  notes.value.push(note);
  newNote.value = "";
  error.value = "";
  toggleModal(false);
};

const deleteNote = (e, id) => {
  e.preventDefault();
  const confirmed = confirm("Are you sure you want to delete this note?");
  if (!confirmed) return;
  notes.value = notes.value.filter(note => note.id !== id);
};

const toggleModal = async (bool) => {
  showModal.value = bool
  await nextTick(); // wait for the DOM to update
  if (bool) textAreaRef.value.focus();
  console.log(textAreaRef.value);
};

</script>

<style scoped>
main {
  height: 100vh;
  width: 100vw;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

h1 {
  font-weight: bold;
  margin-bottom: 25px;
  font-size: 75px;
}

header button {
  border: none;
  padding: 10px;
  width: 50px;
  height: 50px;
  cursor: pointer;
  background-color: rgb(196, 196, 196);
  color: black;
  border-radius: 100%;
  font-size: 20px;
}

.modal button {
  padding: 10px 20px;
  font-size: 20px;
  width: 100%;
  background-color: violet;
  border: none;
  color: white;
  cursor: pointer;
  margin-top: 10px;
}

.cards-container {
  display: flex;
  flex-wrap: wrap;
}

.card {
  width: 225px;
  height: 225px;
  background-color: crimson;
  color: white;
  padding: 10px;
  border-radius: 15px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  margin-right: 20px;
  margin-bottom: 20px;
}

.date {
  font-size: 12px;
  font-weight: bold;
}

.container {
  max-width: 1000px;
  padding: 10px;
  margin: auto;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.77);
  z-index: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal {
  width: 750px;
  background-color: white;
  border-radius: 10px;
  padding: 30px;
  position: relative;
  display: flex;
  flex-direction: column;
}

.modal .close {
  background-color: crimson;
  margin-top: 7px;
}

.error {
  background-color: crimson;
  color: whitesmoke;
  padding: 10px;
  margin-bottom: 10px;
  font-size: 20px;
  font-weight: bold;
  text-align: center;
}

.main-text {
  overflow-wrap: break-word;
}
</style>