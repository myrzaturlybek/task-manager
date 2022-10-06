<template>
  <v-container>
    <draggable
      id="board"
      class="row pa-4"
      v-model="columns"
      group="columns"
      ghost-class="ghost"
      draggable=".isDraggable"
      :delay="200"
      :delay-on-touch-only="true"
    >
      <div
        class="cardColumm mr-3 isDraggable widthColumn"
        v-for="(column, indexColumn) in columns"
        :key="indexColumn"
      >
        <v-card-text class="mb-0 cardColummHeader hoverGrab">
          <p class="title mb-0">{{ column.name }}</p>
        </v-card-text>
        <v-card-actions class="pa-3" v-if="!column.isAddingCard">
          <v-btn block @click="addCardDialogShow(column)" depressed outlined>
            <v-icon>mdi-plus</v-icon> CREATE
          </v-btn>
        </v-card-actions>

        <draggable
          v-model="column.cards"
          group="cards"
          @start="drag = true"
          @end="drag = false"
          :list="column.cards"
          ghost-class="ghost"
          class="v-card__text minHeight scrollBar cardColummContent"
          :id="'idColumn' + column.id"
          :delay="200"
          :delay-on-touch-only="true"
        >
          <v-flex v-for="card in column.cards" :key="card.id">
            <v-card
              class="mb-2 hoverGrab overflow-hidden"
              @mouseover="card.hover = true"
              @mouseleave="card.hover = false"
              @click="onShowTask(card)"
              v-if="!card.editable"
              tile
            >
              <v-card-title>
                <v-btn text class="pa-0">
                  {{ card.name }}
                </v-btn>
                <v-spacer></v-spacer>
              </v-card-title>
              <v-card-text>
                <div class="text-truncate" v-if="card.description">
                  {{ card.description }}
                </div>
              </v-card-text>
            </v-card>
          </v-flex>
        </draggable>
      </div>
    </draggable>

    <v-dialog v-model="dialogShowTask" persistent max-width="1000px">
      <v-card>
        <div v-if="Object.keys(selectedTaskShow).length > 0">
          <v-card-text>
            <v-text-field
              v-show="selectedTaskShow.new || selectedTaskShow.edit"
              v-model="selectedTaskShow.name"
              name="taskTitle"
              label="Title"
              counter="40"
            >
            </v-text-field>
            <v-card-title
              v-show="!selectedTaskShow.new && !selectedTaskShow.edit"
            >
              <span class="headline">{{ selectedTaskShow.name }}</span>
            </v-card-title>
          </v-card-text>

          <v-card-text class="pb-0">
            <v-textarea
              v-show="selectedTaskShow.new || selectedTaskShow.edit"
              v-model="selectedTaskShow.description"
              solo
              name="input-7-4"
              label="Description"
            ></v-textarea>
            <p v-show="!selectedTaskShow.new && !selectedTaskShow.edit">
              {{ selectedTaskShow.description }}
            </p>
            <p class="font-italic">#{{ selectedTaskShow.id }}</p>
            <p class="red--text">{{ dialogError }}</p>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="darken-1" @click="onCloseShowTask">Done</v-btn>
            <v-btn
              v-show="!selectedTaskShow.new && !selectedTaskShow.edit"
              @click="editTask"
            >
              <v-icon>mdi-pencil</v-icon>
            </v-btn>
            <v-btn
              v-show="!selectedTaskShow.new"
              @click="deleteTask()"
              color="red"
            >
              <v-icon color="white">mdi-delete</v-icon>
            </v-btn>
          </v-card-actions>
        </div>
        <div v-else>
          <v-card-text>
            Loading...
            <v-progress-linear indeterminate class="mb-0"></v-progress-linear>
          </v-card-text>
        </div>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import draggable from "vuedraggable";
export default {
  components: {
    draggable,
  },
  data() {
    return {
      lastCardId: 1,
      columns: [
        {
          id: 1,
          name: "TO DO",
          cards: [
            {
              name: "Test card",
              description:
                "Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s",
              id: "1",
              hover: false,
            },
          ],
        },
        {
          id: 2,
          name: "IN PROGRESS",
          cards: [],
        },
        {
          id: 3,
          name: "DONE",
          cards: [],
        },
        {
          id: 4,
          name: "PROD",
          cards: [],
        },
      ],
      dialogShowTask: false,
      selectedTaskShow: {},
      isAddingCard: false,
      cardCreating: {},
      isAddingColumn: false,
      nameColumn: "",
      dialogError: "",
      newCardColumn: {},

      //Testes
      testCheckList: false,
    };
  },
  methods: {
    onShowTask(element) {
      this.$router.push({ query: { task: element.id } });
      this.dialogShowTask = true;
      setTimeout(() => {
        this.selectedTaskShow = element;
      }, 500);
    },
    deleteTask() {
      this.columns.forEach((column) => {
        column.cards = column.cards.filter((card) => {
          return card.id !== this.selectedTaskShow.id;
        });
      });
      this.selectedTaskShow = {};
      this.dialogShowTask = false;
    },
    editTask() {
      this.selectedTaskShow.edit = true;
      this.dialogShowTask = false;
      this.onShowTask(this.selectedTaskShow);
    },
    onCloseShowTask() {
      if (
        this.selectedTaskShow.name != "" &&
        this.selectedTaskShow.name.length <= 40
      ) {
        if (this.selectedTaskShow.new) {
          this.addCard(this.newCardColumn);
        }
        this.$router.push({ query: {} });
        this.selectedTaskShow.edit = false;
        this.selectedTaskShow.new = false;
        this.dialogShowTask = false;
        this.selectedTaskShow = {};
        this.dialogError = "";
      } else {
        this.dialogError = "Enter task title";
      }
    },
    addCard(column) {
      this.lastCardId = this.lastCardId + 1;
      column.cards.push({
        id: this.selectedTaskShow.id,
        name: this.selectedTaskShow.name,
        description: this.selectedTaskShow.description,
      });
    },
    addCardDialogShow(column) {
      this.newCardColumn = column;
      this.dialogShowTask = true;
      this.selectedTaskShow = {
        new: true,
        name: "",
        description: "",
        id: String(this.lastCardId + 1),
        hover: false,
      };
    },
  },
  watch: {
    $route(to) {
      // react to route changes...
      if (!("task" in to.query)) {
        this.onCloseShowTask();
      }
    },
  },
};
</script>


<style>
#board {
  height: 100%;
  position: absolute;
  top: 0px;
  left: 0px;
  right: 0px;
  box-sizing: border-box;
  padding: 10px;
  align-items: flex-start;
  overflow-x: auto;
  flex-wrap: nowrap;
}

.borderText {
  border: 1px solid;
}

.minHeight {
  min-height: 350px;
}

.ghost,
.sortable-chesen {
  opacity: 0.5;
}

.draggable {
  border: 1px solid;
}

.widthColumn {
  width: 300px !important;
  display: flex;
  flex: 0 0 auto;
}

.cardColumm {
  max-height: 100%;
  flex-direction: column;
  display: flex;
  flex: 0 0 auto;
  align-items: stretch !important;
  box-sizing: border-box;
  box-shadow: 1px 1px 3px -2px #000;
  background: #b1b1b1;
}

.cardColummHeader {
  flex: 0 0 auto;
  position: relative;
}

.cardColummContent {
  flex: 1 1 auto;
  margin-bottom: 0;
  overflow-y: auto;
  overflow-x: hidden;
  z-index: 1;
  min-height: 0;
}

.draggable {
  min-height: 150px;
  border: 1px solid;
  display: inline;
}

.hoverPointer:hover {
  cursor: pointer;
}

.hoverGrab:hover {
  cursor: grab;
}

.button {
  margin-top: 35px;
}
.flip-list-move {
  transition: transform 0.5s;
}
.no-move {
  transition: transform 0s;
}

.list-group {
  min-height: 20px;
}
.list-group-item {
  cursor: move;
}
.list-group-item i {
  cursor: pointer;
}

.scrollBar::-webkit-scrollbar-track {
  -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
  border-radius: 5px;
  background-color: rgba(0, 0, 0, 0.1);
}

.scrollBar::-webkit-scrollbar {
  width: 6px;
  background-color: rgba(0, 0, 0, 0.1);
}

.scrollBar::-webkit-scrollbar-thumb {
  border-radius: 5px;
  -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
  background-color: #555;
}
</style>