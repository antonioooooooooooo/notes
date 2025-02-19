<template>
  <div class="page-wrapper">
    <button @click="addNote" class="add-note-button">Add Note</button>
    <Note
      v-for="note in notes"
      :key="note.id"
      :content="note.content"
      :posX="note.posX"
      :posY="note.posY"
      :height="note.height"
      :width="note.width"
      :id="note.id"
      :title="note.title"
      :color="note.color"
      @update:posX="(newX) => updateNote(note.id, 'posX', newX)"
      @update:posY="(newY) => updateNote(note.id, 'posY', newY)"
      @update:title="(newTitle) => updateNote(note.id, 'title', newTitle)"
      @update:content="
        (newContent) => updateNote(note.id, 'content', newContent)
      "
      @update:width="(newWidth) => updateNote(note.id, 'width', newWidth)"
      @update:height="(newHeight) => updateNote(note.id, 'height', newHeight)"
      @update:color="(newColor) => updateNote(note.id, 'color', newColor)"
      @delete-note="deleteNote(note.id)"
    />
  </div>
</template>

<script>
import Note from "./Note.vue";

export default {
  components: {
    Note,
  },
  data() {
    return {
      notes: [],
    };
  },
  methods: {
    addNote() {
      let posX = window.innerWidth / 2,
        posY = window.innerHeight / 2;

      while (
        !this.notes.every((note) => note.posX !== posX && note.posY !== posY)
      ) {
        posX += 20;
        posY += 20;
      }

      this.notes.push({
        posX,
        posY,
        height: 200,
        width: 200,
        content: "",
        id: "note" + (this.notes.length + 1).toString(),
        title: "Note " + (this.notes.length + 1).toString(),
        color: null,
      });
    },
    updateNote(id, key, value) {
      const index = this.notes.findIndex((note) => note.id === id);
      if (index !== -1) {
        this.notes.splice(index, 1, { ...this.notes[index], [key]: value });
      }
    },
    deleteNote(id) {
      this.notes = this.notes.filter((note) => note.id !== id);
    },
  },
  watch: {
    notes: {
      handler(newNotes) {
        localStorage.setItem("notes", JSON.stringify(newNotes));
      },
      deep: true,
    },
  },
  mounted() {
    const localStorageNotes = localStorage.getItem("notes");
    if (localStorageNotes) {
      this.notes = JSON.parse(localStorageNotes);
    }
  },
};
</script>

<style scoped>
.page-wrapper {
  display: flex;
  justify-content: center;
  padding: 80px 0 0;
  position: relative;
  background-color: #faaca8;
  background-image: linear-gradient(19deg, #faaca8 0%, #ddd6f3 100%);
  width: 100%;
  height: 100%;
}

.add-note-button {
  align-items: center;
  appearance: none;
  background-image: radial-gradient(
    100% 100% at 100% 0,
    #5adaff 0,
    #5468ff 100%
  );
  border: 0;
  border-radius: 6px;
  box-shadow: rgba(45, 35, 66, 0.4) 0 2px 4px,
    rgba(45, 35, 66, 0.3) 0 7px 13px -3px, rgba(58, 65, 111, 0.5) 0 -3px 0 inset;
  box-sizing: border-box;
  color: #fff;
  cursor: pointer;
  display: inline-flex;
  font-family: "JetBrains Mono", monospace;
  height: 48px;
  justify-content: center;
  line-height: 1;
  list-style: none;
  overflow: hidden;
  padding-left: 16px;
  padding-right: 16px;
  position: relative;
  text-align: left;
  text-decoration: none;
  transition: box-shadow 0.15s, transform 0.15s;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  white-space: nowrap;
  will-change: box-shadow, transform;
  font-size: 18px;
}

.add-note-button:focus {
  box-shadow: #3c4fe0 0 0 0 1.5px inset, rgba(45, 35, 66, 0.4) 0 2px 4px,
    rgba(45, 35, 66, 0.3) 0 7px 13px -3px, #3c4fe0 0 -3px 0 inset;
}

.add-note-button:hover {
  box-shadow: rgba(45, 35, 66, 0.4) 0 4px 8px,
    rgba(45, 35, 66, 0.3) 0 7px 13px -3px, #3c4fe0 0 -3px 0 inset;
  transform: translateY(-2px);
}

.add-note-button:active {
  box-shadow: #3c4fe0 0 3px 7px inset;
  transform: translateY(2px);
}
</style>
