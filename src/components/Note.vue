<template>
  <div
    :id="id"
    class="note"
    :style="{
      height: localHeight + 'px',
      width: localWidth + 'px',
      backgroundColor: darkenColor(localColor),
    }"
  >
    <div :id="id + 'draggable'" class="draggable-header">
      <div v-if="!editingTitle">{{ localTitle }}</div>
      <input
        v-if="editingTitle"
        type="text"
        class="note-title-input"
        v-model="localTitle"
        @input="updateTitle"
        @keydown.enter="toggleEditingTitle"
      />
      <div class="images-wrapper">
        <div>
          <input class="color-picker" type="color" @input="onColorChange" />
        </div>
        <img
          v-if="!editingTitle"
          class="pencil-image"
          src="https://www.svgrepo.com/show/29974/pencil.svg"
          @click="toggleEditingTitle"
        />
        <img
          v-if="editingTitle"
          class="save-image"
          src="https://www.svgrepo.com/show/12183/save.svg"
          @click="toggleEditingTitle"
        />
        <img
          class="trash-can-image"
          src="https://www.svgrepo.com/show/132300/trash-can-with-cover-from-side-view.svg"
          @click="$emit('delete-note')"
        />
      </div>
    </div>
    <textarea
      v-model="localContent"
      class="note-content"
      @input="updateContent"
      :style="{ backgroundColor: localColor }"
    />
    <div class="resizer" @mousedown="initResize"></div>
  </div>
</template>

<script>
export default {
  props: {
    id: String,
    posX: Number,
    posY: Number,
    height: Number,
    width: Number,
    title: String,
    content: String,
    color: String,
  },
  data() {
    return {
      localContent: this.content,
      localHeight: this.height,
      localWidth: this.width,
      localTitle: this.title,
      localColor: this.color || "#f2f2f2",
      editingTitle: false,
    };
  },
  methods: {
    updateContent() {
      this.$emit("update:content", this.localContent);
    },
    updateSize() {
      this.$emit("update:height", this.localHeight);
      this.$emit("update:width", this.localWidth);
    },
    updateTitle() {
      this.$emit("update:title", this.localTitle);
    },
    dragElement(noteElement) {
      let pos1 = 0,
        pos2 = 0,
        pos3 = 0,
        pos4 = 0;

      const noteDraggableHeaderElement = document.getElementById(
        this.id + "draggable"
      );

      const dragMouseDown = (e) => {
        e.preventDefault();
        pos3 = e.clientX;
        pos4 = e.clientY;
        document.onmouseup = closeDragElement;
        document.onmousemove = elementDrag;
      };

      noteDraggableHeaderElement.onmousedown = dragMouseDown;

      const elementDrag = (e) => {
        e.preventDefault();
        pos1 = pos3 - e.clientX;
        pos2 = pos4 - e.clientY;
        pos3 = e.clientX;
        pos4 = e.clientY;

        const newX = noteElement.offsetLeft - pos1;
        const newY = noteElement.offsetTop - pos2;

        noteElement.style.left = newX + "px";
        noteElement.style.top = newY + "px";

        this.$emit("update:posX", newX);
        this.$emit("update:posY", newY);
      };

      const closeDragElement = () => {
        document.onmouseup = null;
        document.onmousemove = null;
      };
    },
    resize(e) {
      const noteElement = document.getElementById(this.id);

      this.localWidth =
        e.clientX - noteElement.offsetLeft + noteElement.offsetWidth / 2;
      this.localHeight =
        e.clientY - noteElement.offsetTop + noteElement.offsetHeight / 2;

      this.updateSize();
    },
    initResize(e) {
      e.preventDefault();
      document.onmousemove = this.resize;
      document.onmouseup = this.stopResize;
    },
    stopResize() {
      document.onmousemove = null;
      document.onmouseup = null;
    },
    toggleEditingTitle() {
      this.editingTitle = !this.editingTitle;
      this.$nextTick(() => {
        if (this.editingTitle) {
          this.$el.querySelector(".note-title-input").focus();
        }
      });
    },
    onColorChange(e) {
      this.localColor = e.target.value;
      this.$emit("update:color", this.localColor);
    },
    darkenColor(color) {
      let colorHex = color.replace("#", "");
      let r = parseInt(colorHex.substring(0, 2), 16);
      let g = parseInt(colorHex.substring(2, 4), 16);
      let b = parseInt(colorHex.substring(4, 6), 16);

      r = Math.max(0, r - r * 0.2);
      g = Math.max(0, g - g * 0.2);
      b = Math.max(0, b - b * 0.2);

      return `rgb(${r}, ${g}, ${b})`;
    },
  },
  mounted() {
    const noteElement = document.getElementById(this.id);
    noteElement.style.top = this.posY + "px";
    noteElement.style.left = this.posX + "px";

    this.dragElement(noteElement);
  },
};
</script>

<style scoped>
.note {
  position: absolute;
  transform: translate(-50%, -50%);
  border-radius: 6px;
  box-shadow: rgba(0, 0, 0, 0.35) 0px 5px 15px;
  overflow: hidden;
}

.draggable-header {
  cursor: move;
  user-select: none;
  height: 30px;
  display: flex;
  align-items: center;
  padding: 0 8px;
  font-family: "JetBrains Mono", monospace;
  font-weight: 700;
  justify-content: space-between;
}

.note-content {
  height: fit-content;
  border: none;
  resize: none;
  outline: none;
  width: 100%;
  height: 100%;
  padding: 6px;
  box-sizing: border-box;
  font-family: "JetBrains Mono", monospace;
  overflow: hidden;
  font-size: 16px;
}

.resizer {
  width: 15px;
  height: 15px;
  background: gray;
  position: absolute;
  bottom: 0;
  right: 0;
  cursor: se-resize;
  border-bottom-right-radius: 6px;
}
.pencil-image,
.save-image {
  height: 16px;
  width: 16px;
  cursor: pointer;
}

.trash-can-image {
  height: 16px;
  width: 16px;
  cursor: pointer;
}

.note-title-input {
  width: 70%;
  font-family: "JetBrains Mono", monospace;
  font-weight: 700;
  font-size: 16px;
  box-sizing: border-box;
  border: 1px solid gray;
  padding-left: 4px;
  border-radius: 4px;
}

.color-picker {
  height: 16px;
  width: 16px;
  cursor: pointer;
}

.images-wrapper {
  display: flex;
  gap: 12px;
  align-items: center;
}
</style>
