<template>
  <div class="hiring-details-main kanban">
    <div class="page-heading d-flex align-items-center">
      <img
        @click="$router.go(-1)"
        class="pointer"
        src="../assets/backarrow.svg"
        alt=""
      />

      <div class="main-head ml-4">
        <span>{{ $route.params.name }} > Kanban View</span>
      </div>
    </div>
    <b-button v-b-modal.column-modal> Add New Column </b-button>
    <div class="k-container mt-3">
      <div class="d-flex kcard-maincontainer mt-2">
        <draggable
          v-model="columns"
          :options="{ group: { name: 'columns' } }"
          class="d-flex"
        >
          <div
            class="kcard-subcontainer mr-2"
            v-for="(column, index) in columns"
            :key="column.id"
          >
            <div
              :style="`background:${column.background}`"
              class="kcard-head text-center relativeclass"
            >
              {{ column.name }}

              <div class="noofcands">{{ column.tasks.length }}</div>
            </div>
            <div class="kcard-sub-subcontainer">
              <button @click="addNewTask(index)">
                <i class="fas fa-plus"></i>Add Task
              </button>
              <draggable
                v-model="column.tasks"
                :options="{ group: { name: 'tasks' } }"
              >
                <div v-for="(task, index) in column.tasks" :key="index">
                  <div class="kanbancards mt-3 relativeclass">
                    <div
                      :style="`background:${column.background}`"
                      class="kcard-unscheduled-colorbar ml-1 mr-1"
                    ></div>
                    <div class="ml-3">
                      <div
                        class="d-flex justify-content-between align-items-center"
                      >
                        <div class="profilepic">{{ task.name }}</div>
                      </div>
                      <i
                        @click="editTask(column, task)"
                        class="fas fa-pen mr-2 pointer"
                      ></i>
                      <i
                        @click="deleteTask(column, task)"
                        class="fas fa-trash pointer"
                      ></i>
                      <div
                        class="d-flex justify-content-between align-items-center"
                      >
                        <div class="d-flex align-items-center">
                          <div class="kanban-viewcandtext ml-2 pointer">
                            {{ task.description }}
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </draggable>
            </div>
          </div>
        </draggable>
      </div>
    </div>

    <b-modal
      id="task-modal"
      centered
      hide-footer
      hide-header
      no-close-on-backdrop
    >
      <div class="modalheading">
        {{ kanbanStatus === "Add" ? "Add New Task" : "Edit New Task" }}
      </div>
      <div class="xMarkCancel" @click="$bvModal.hide('task-modal')">
        <img src="../assets/xmark.svg" alt="" />
      </div>
      <hr />
      <b-row
        ><b-col>
          <div class="topDiv mt-3 relativeclass">
            <label class="fieldsetStyle"
              >Task Name &nbsp;<span class="asterik">*</span></label
            >
            <b-form-input
              v-model="name"
              class="inputField"
              id="input-1"
              type="text"
              placeholder="Enter here"
              autofocus
            ></b-form-input>
          </div>
        </b-col>
      </b-row>

      <div>
        <div class="topDiv mt-3 mb-2">
          <label class="fieldsetStyle">Description</label>
          <b-form-textarea
            v-model="description"
            id="textarea-large"
            class="inputField"
            size="lg"
            rows="3"
            max-rows="6"
          ></b-form-textarea>
        </div>
      </div>
      <div class="d-flex align-items-center justify-content-end">
        <b-button
          class="mb-1 pl-3 pr-3"
          @click="kanbanStatus === 'Add' ? saveTask() : saveEditTask()"
        >
          {{ kanbanStatus === "Add" ? "Save" : "Update" }}
        </b-button>
      </div>
    </b-modal>
    <b-modal
      id="column-modal"
      centered
      hide-footer
      hide-header
      no-close-on-backdrop
      @show="resetColumnModal"
      @hidden="resetColumnModal"
    >
      <div class="modalheading">Add New Column</div>
      <div class="xMarkCancel" @click="$bvModal.hide('column-modal')">
        <img src="../assets/xmark.svg" alt="" />
      </div>

      <hr />
      <b-row
        ><b-col>
          <div class="topDiv mt-3 relativeclass mb-2">
            <label class="fieldsetStyle"
              >Column Name &nbsp;<span class="asterik">*</span></label
            >
            <b-form-input
              v-model="columnName"
              class="inputField"
              id="input-1"
              type="text"
              placeholder="Enter here"
              autofocus
            ></b-form-input>
          </div>
        </b-col>
      </b-row>

      <div class="d-flex align-items-center justify-content-end">
        <b-button class="mb-1 pl-3 pr-3" @click="saveColumn()">Add</b-button>
      </div>
    </b-modal>
  </div>
</template>

<script>
import draggable from "vuedraggable";
import { useStore } from "../store";

export default {
  name: "KanbanView",
  components: {
    draggable,
  },
  data() {
    return {
      name: "",
      description: "",
      columnName: "",
      kanbanStatus: "",
      kanbanIndex: null,
      columnEditValues: {},
      taskEditValues: {},
      columns: [],
    };
  },
  created() {
    // Retrieve the Kanban board data from the Pinia store and assign column array, or assign an empty array if it doesn't exist
    const { columns } = this.kanbanStore.kanbanData.find(
      (i) => i.id == this.$route.params.id
    ) || { columns: [] };

    this.columns = columns;
  },
  setup() {
    const kanbanStore = useStore();
    return { kanbanStore };
  },
  methods: {
    /**
     * Sets the index of the currently selected column and sets the kanbanStatus to "Add",
     * then shows the "task-modal" BootstrapVue modal.
     *
     * @param {Number} index - The index of the currently selected column.
     */
    addNewTask(index) {
      // Set the index of the currently selected column to the component's kanbanIndex property
      this.kanbanIndex = index;

      // Set the kanbanStatus property to "Add" to indicate that a new task is being added
      this.kanbanStatus = "Add";

      // Show the "task-modal" BootstrapVue modal
      this.$bvModal.show("task-modal");
    },

    /**
     * Saves a new task to the specified column index of the kanban board.
     */
    saveTask() {
      // Find the index of the kanban board based on the route parameter ID
      const columnIndex = this.kanbanStore.kanbanData.findIndex(
        (i) => i.id == this.$route.params.id
      );
      // Call the savekanbanTask action to save the new task, passing in the task details and the column index
      this.kanbanStore.savekanbanTask({
        index: this.kanbanIndex, // The index of the new task in the column's tasks array
        name: this.name, // The name of the new task
        description: this.description, // The description of the new task
        columnIndex, // The index of the column to add the new task to
      });

      // Hide the modal and reset the form values
      this.$bvModal.hide("task-modal");
      this.resetModal();
    },
    saveColumn() {
      // Hide the modal dialog
      this.$bvModal.hide("column-modal");

      // Find the index of the Kanban board based on the route parameter
      const columnIndex = this.kanbanStore.kanbanData.findIndex(
        (i) => i.id == this.$route.params.id
      );

      // Create a new column object
      const newColumn = {
        // Set the ID of the new column to be one greater than the current number of columns
        id: this.kanbanStore.kanbanData[columnIndex].columns.length + 1,
        // Set the name of the new column to the name entered by the user
        name: this.columnName,
        // Generate a random background color for the new column
        background: "#" + Math.random().toString(16).substr(-6),
        // Initialize the tasks array for the new column to be empty
        tasks: [],
      };

      // Call the savekanbanColumn action of the Kanban store to save the new column
      this.kanbanStore.savekanbanColumn({
        id: this.$route.params.id,
        newColumn,
      });
    },

    /**
     * Sets the modal state to "Edit" and initializes the modal form fields with the current task data.
     *
     * @param {Object} column - The column object that the task belongs to.
     * @param {Object} task - The task object to edit.
     */
    editTask(column, task) {
      // Set the kanbanStatus to "Edit" so that the modal is rendered with the appropriate header.
      this.kanbanStatus = "Edit";

      // Set the modal form fields to the current task data.
      this.name = task.name;
      this.description = task.description;

      // Set the columnEditValues and taskEditValues variables so that we can use them in the saveTask method.
      this.columnEditValues = column;
      this.taskEditValues = task;
      // Show the modal.
      this.$bvModal.show("task-modal");
    },
    saveEditTask() {
      //This line finds the index of the Kanban board in the kanbanData array of the kanbanStore that matches the ID passed in the route parameters.
      const idIndex = this.kanbanStore.kanbanData.findIndex(
        (i) => i.id == this.$route.params.id
      );

      // This line finds the index of the Kanban column in the columns array of the Kanban board found in the previous step that matches the ID of the column being edited.
      const columnIndex = this.kanbanStore.kanbanData[
        idIndex
      ].columns.findIndex((i) => i.id === this.columnEditValues.id);

      // This line finds the index of the Kanban task in the tasks array of the Kanban column found in the previous step that matches the ID of the task being edited.
      const taskIndex = this.kanbanStore.kanbanData[idIndex].columns[
        columnIndex
      ].tasks.findIndex((j) => j.id === this.taskEditValues.id);

      // This line calls the editKanbanTask action of the kanbanStore with the appropriate parameters to update the task with the new name and description.
      this.kanbanStore.editKanbanTask({
        idIndex,
        columnIndex,
        taskIndex,
        name: this.name,
        description: this.description,
      });

      // This line clears the form inputs to prepare for a new task to be added.
      this.resetModal();

      // This line hides the edit task modal once the changes have been saved.
      this.$bvModal.hide("task-modal");
    },
    // Deletes a task from the Kanban board.
    deleteTask(column, task) {
      // Find the index of the Kanban board with the ID from the route params.
      const idIndex = this.kanbanStore.kanbanData.findIndex(
        (i) => i.id == this.$route.params.id
      );
      // Find the index of the column in the Kanban board.
      const columnIndex =
        this.kanbanStore.kanbanData[idIndex].columns.indexOf(column);

      // Find the index of the task in the column.
      const taskIndex =
        this.kanbanStore.kanbanData[idIndex].columns[columnIndex].tasks.indexOf(
          task
        );
      // This line calls the editKanbanTask action of the kanbanStore to delete the task
      this.kanbanStore.deleteKanbanTask({ idIndex, columnIndex, taskIndex });
    },
    resetColumnModal() {
      this.columnName = "";
    },
    resetModal() {
      this.name = "";
      this.description = "";
    },
  },
};
</script>

<style>
.final-select-options-kanban {
  color: #545454;
  font-weight: 500;
  font-size: 14px;
}
.final-select-modal-text-kanban {
  color: #545454;
  font-weight: 400;
  font-size: 14px;
}
.radio-margins {
  margin-top: 4px;
}
.feedbk-round-head-kanban {
  font-weight: 500;
  font-size: 14px;
  color: #545454;
}
.panel-un {
  margin-top: 10px;
  margin-left: 15px;
  height: 55px;
  width: 400px;
  background: #e7e8f5;
  border-radius: 3px;
  padding: 5px;
}
.status-text {
  font-weight: 400;
  font-size: 12px;
}
.panel-name {
  font-weight: 400;
  color: #72738e;
  font-size: 14px;
}
.panel-status {
  margin-top: 10px;
  margin-left: 15px;
  height: 55px;
  width: 124px;
  background: #e7e8f5;
  border-radius: 3px;
  padding: 5px;
}
.custom-control-label::before {
  margin-top: 4px;
}
.orcolclass {
  display: flex;
  justify-content: center;
  align-items: center;
}
.qb-dropdown-option-sing:nth-child(even) {
  background: #fdfdfd;
  padding: 10px 30px;
}

.qb-dropdown-option-sing:nth-child(odd) {
  background: #f4f4f4;
  padding: 10px 30px;
}
.folder-name {
  font-size: 14px;
  font-weight: 400;
  color: #6d6d6d;
}
.folder-list {
  max-height: 165px;
  overflow: auto;
}
.recsearchicon-pos {
  position: absolute;
  top: 5px;
  left: 5px;
}
.recinput .form-control {
  padding: 0.375rem 1.3rem !important;
}
.kanban-cj-filter-modal {
  padding: 10px;
  background: #ffffff;
  border: 0.7px solid rgba(114, 115, 142, 0.5);
  box-sizing: border-box;
  box-shadow: 0px 4px 14px rgba(0, 0, 0, 0.12);
  border-radius: 5px;
  z-index: 6;
  position: absolute;
  width: 100%;
  max-height: 200px;
  min-height: auto;
  overflow: auto;
}
.folder-bar-head {
  font-weight: var(--weight-M);
  font-size: 14px;
  color: #696969;
}
.inputimage-false {
  position: absolute;
  width: 100%;
  height: 50%;
  left: 0;
  top: 0;
}
.orclass {
  font-weight: 400;
  font-size: 14px;
  /* identical to box height */
  color: #c4c4c4;
}
.inputimage {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
}
input[type="file"] {
  opacity: 0;
}
.kcard-colorbar {
  width: 4px;
  height: 130px;
  border-radius: 2px;
  position: absolute;
  background: #ffc145;
  left: 0px;
  top: 2px;
}
.kcard-unscheduled-colorbar {
  width: 4px;
  height: 95px;
  border-radius: 2px;
  position: absolute;
  left: 0px;
  top: 2px;
}
.kselectedcard-colorbar {
  width: 4px;
  height: 130px;
  border-radius: 2px;
  position: absolute;
  left: 0px;
  top: 2px;
  background: #038e00;
}
.knicard-colorbar {
  width: 4px;
  height: 130px;
  border-radius: 2px;
  position: absolute;
  left: 0px;
  top: 2px;
  background: #a9a9a9;
}
.krejectedcard-colorbar {
  width: 4px;
  height: 130px;
  border-radius: 2px;
  position: absolute;
  left: 0px;
  top: 2px;
  background: #e04a58;
}
.konholdcard-colorbar {
  width: 4px;
  height: 130px;
  border-radius: 2px;
  position: absolute;
  left: 0px;
  top: 2px;
  background: #ffc145;
}
.kcard-emailid {
  font-weight: 400;
  font-size: 13px;
  color: #72738e;
}
.progressbtn {
  background: rgba(73, 190, 155, 0.51);
  border-radius: 3px;
  font-weight: 500;
  font-size: 12px;
  color: #038e00;
  padding: 3px 12px;
}
.assesmentongoing {
  background: #ffc145;
  border-radius: 3px;
  font-weight: 500;
  font-size: 12px;
  color: white;
  padding: 3px 12px;
  opacity: 0.5;
}
.assessmentyet {
  background: #d90606;
  border-radius: 3px;
  font-weight: 500;
  font-size: 12px;
  color: white;
  padding: 3px 12px;
  opacity: 0.5;
}
.kcard-cand-name {
  font-weight: 600;
  font-size: 14px;
  color: #72738e;
}
.schedulebtn {
  background: #72738e;
  border-radius: 3px !important;
  color: #ffffff;
  /* padding: 4px 12px; */
  padding: 4px 7px;
  text-align: center;
  font-weight: 500;
  font-size: 12px;
}
.schedulebtn:hover {
  background: #72738e;
  border-radius: 3px !important;
  color: #ffffff;
  text-align: center;
  /* padding: 4px 12px; */
  padding: 4px 7px;
  font-weight: 500;
  font-size: 12px;
}
.view-feedbtn {
  font-weight: 400;
  /* padding: 4px 12px; */
  padding: 4px 7px;
  font-size: 12px;
  color: #72738e;
  background: #fcfcfc;
  border: 1px solid #72738e;
  box-sizing: border-box;
  text-align: center;
  border-radius: 3px !important;
}
.profilepic {
  height: 30px;
  width: 30px;
  /* border: 1px solid red; */
  border-radius: 15px;
}
.noofcands {
  position: absolute;
  right: 2px;
  bottom: 4px;
  font-weight: 500;
  font-size: 14px;
  background: #ffffff;
  color: #72738e;
  padding: 3px 10px;
  border-radius: 4px;
}
.kselectedcard-head {
  color: #ffffff;

  font-weight: 600;
  font-size: 16px;
  border-radius: 5px;
  padding: 5px;
  background: #038e00;
}
.konholdcard-head {
  color: #ffffff;

  font-weight: 600;
  font-size: 16px;
  border-radius: 5px;
  padding: 5px;
  background: #ffc145;
}
.knicard-head {
  color: #ffffff;

  font-weight: 600;
  font-size: 16px;
  border-radius: 5px;
  padding: 5px;
  background: #a9a9a9;
}
.krejcard-head {
  color: #ffffff;

  font-weight: 600;
  font-size: 16px;
  border-radius: 5px;
  padding: 5px;
  background: #e04a58;
}
.kcard-head {
  color: #ffffff;
  font-weight: 600;
  font-size: 16px;
  border-radius: 5px;
  padding: 5px;
}
.kanbancards {
  background: #ffffff;
  box-shadow: 0px 4px 12px rgba(196, 196, 196, 0.5);
  border-radius: 5px;
  padding: 10px;
  height: 100px;
}
.kcard-maincontainer {
  display: flex;
  overflow: auto;
}
.k-container {
  background: #ffffff;
  border-radius: 8px;
  height: calc(100vh - 150px);
  /* padding: 15px; */
}
.kcard-subcontainer {
  background: #f3f3f3;
  padding: 5px;
  border-radius: 5px;
  height: calc(100vh - 80px);

  flex-grow: 0;
  flex-shrink: 0;
  flex-basis: 320px;
}
.kcard-sub-subcontainer {
  height: calc(100vh - 130px);
  overflow: auto;
}
.sched .btn-outline-secondary:hover {
  font-weight: 400;
  /* padding: 4px 12px; */
  padding: 4px 7px;
  text-align: center;
  font-size: 12px;
  color: #72738e;
  background-color: #fcfcfc;
  border: 1px solid #72738e;
  box-sizing: border-box;
  border-radius: 3px !important;
  border-color: unset;
}
.sched .btn-outline-secondary:not(:disabled):not(.disabled):active,
.btn-outline-secondary:not(:disabled):not(.disabled).active,
.show > .btn-outline-secondary.dropdown-toggle {
  background-color: #fcfcfc;
  border-radius: 3px;
  border-color: unset;
  color: #72738e;
}

.btn-outline-secondary:not(:disabled):not(.disabled):active:focus,
.btn-outline-secondary:not(:disabled):not(.disabled).active:focus,
.show > .btn-outline-secondary.dropdown-toggle:focus {
  box-shadow: unset;
}
/* not scheduled btn */
.notschedbluebtn {
  background: rgba(91, 147, 255, 0.24);
  border-radius: 3px;
  font-weight: 500;
  font-size: 12px;
  color: #407bff;
  padding: 3px 12px;
}
</style>

<style>
.feedback-right-container {
  border-radius: 8px;
}
.preview-head-text {
  font-weight: 600;
  font-size: 14px;
  line-height: 21px;
  color: #696969;
}

.fdbck-recodn-container {
  background: #ffffff;
  box-shadow: 0px 0px 4px rgba(0, 0, 0, 0.25);
  border-radius: 0px 0px 4px 4px;
  padding: 25px;
}
.interskilkls-head {
  font-weight: 600;
  font-size: 13px;
  color: #72738e;
  word-break: break-all;
}
.interskills-container {
  background: #ffffff;
  box-shadow: 0px 0px 4px rgba(0, 0, 0, 0.25);
  border-radius: 4px;
  padding: 10px;
}
.fdbck-right-head-container {
  padding: 10px 15px;
  font-weight: 600;
  font-size: 14px;
  color: #ffffff;
  margin: 5px;
  background: rgba(10, 86, 200, 0.5);
  border-radius: 2px;
}
.fdbck-right-detail-container {
  background: #ffffff;
  border: 0.7px solid #e5e5e5;
  box-sizing: border-box;
  box-shadow: 0px 1px 20px rgba(86, 86, 86, 0.2);
  border-radius: 5px;
  height: calc(100vh - 195px);
  overflow: auto;
  margin: 5px 0px;
}
.preview-feedbackform-main-card {
  background: #fbfbfb;
  border: 0.5px solid #e5e5e5;
  box-sizing: border-box;
  box-shadow: 0px 0px 20px rgba(86, 86, 86, 0.2);
  border-radius: 5px;
  padding: 10px;
  width: 300px;
  margin: 15px;
  height: calc(90vh - 140px);
  overflow: auto;
  position: relative;
}
.preview-feedbackform-header h5 {
  font-weight: 600;
  font-size: 14px !important;
  color: #696969;
  padding: 15px;
  padding-bottom: 0;
}
.b-rating.disabled {
  background-color: unset;
}
.ssbox {
  background: #fdfdfd;
  border: 0.3px solid rgba(10, 86, 200, 0.5);
  box-sizing: border-box;
  box-shadow: inset 0px 0px 4px rgba(0, 0, 0, 0.25);
  border-radius: 1px;

  height: 87px;
}
.jd-or-style {
  font-family: "Poppins";
  font-style: normal;
  font-weight: 400;
  font-size: 10px;

  color: #72738e;
}
.jd-panel {
  padding: 7px;
  /* width: 192px;
  height: 34px; */
  left: 915px;
  top: 274px;

  border: 1px solid #72738e;
  border-radius: 5px;
}
.jd-panel-selection-card {
  width: 739px;
  padding: 10px;

  background: #f2f2f2;
}
.jd-panel-selection {
  background: #ffffff;
  box-shadow: 2px 0px 12px rgba(0, 0, 0, 0.08);
  border-radius: 10px;
}
.give-text {
  margin-left: 3px;
  font-weight: 600;
  font-size: 12px;
  line-height: 9px;
  /* identical to box height */

  color: #535461;
}
.b-rating.disabled {
  background-color: unset;
  color: #6c757d;
}
.not-add {
  font-weight: normal;
  font-size: 5px;
  line-height: 7px;

  color: #72738e;
}
.add-notepreview {
  font-weight: normal;
  font-size: 10px;
  line-height: 7px;
  /* identical to box height */
  margin-top: 14px;

  color: #72738e;
}

.hov {
  width: 100px;
  height: 30px;
  background: rgba(255, 0, 0, 0.5) !important;
  box-shadow: 0px 4px 20px rgba(0, 0, 0, 0.18);
  border-radius: 6px;
  font-weight: 500;
  font-size: 14px;
  color: #ffffff;
  border: unset;
  line-height: 15px;
}

.button-previewcust {
  position: absolute;
  top: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  left: 0;
  /* transform: translate(-50%, -50%); */
  z-index: 1;
  height: 100%;
  width: 100%;
  visibility: hidden;

  background: rgba(255, 255, 255, 0.5);
  backdrop-filter: blur(4px);
}

.preview-feedbackform-main-card:hover .button-previewcust {
  visibility: visible;
}

.hov-two {
  top: 10px;
  width: 100px;
  height: 30px;
  background: rgba(10, 86, 200, 0.5) !important;
  border-radius: 6px;
  border: unset;
  font-weight: 500;
  margin-top: 20px;
  font-size: 14px;
  line-height: 15px;
  color: #ffffff;
}
.internal-panelist-text {
  font-family: "Poppins";
  font-style: normal;
  font-weight: 500;
  font-size: 14px;

  color: #72738e;
}
.internal-panelist-text-disable {
  font-family: "Poppins";
  font-style: normal;
  font-weight: 500;
  font-size: 14px;

  color: #c4c4c4;
}
.jdpanel {
  padding: 3px;
  /* width: 192px;
  height: 34px; */
  /* left: 915px;
  top: 274px; */

  border: 1px solid #72738e;
  border-radius: 5px;
}
.jdpanel-disable {
  padding: 3px;
  /* width: 192px;
  height: 34px; */
  /* left: 915px;
  top: 274px; */

  border: 1px solid #c4c4c4;
  border-radius: 5px;
}
.internal-panelist-text {
  font-family: "Poppins";
  font-style: normal;
  font-weight: 500;
  font-size: 14px;

  color: #72738e;
}
.basic-txt {
  margin-bottom: 10px;
  font-weight: 500;
  font-size: 16px;
  line-height: 24px;

  color: #696969;
}
.internal-selected-text {
  font-family: "Poppins";
  font-style: normal;
  font-weight: 500;
  font-size: 12px;
  line-height: 18px;
  /* identical to box height */

  color: #696969;
}
.preview-feedbackform-main-card.cards-form {
  background: #fbfbfb;
  border: 0.5px solid #e5e5e5;
  box-sizing: border-box;
  box-shadow: 0px 0px 20px rgb(86 86 86 / 20%);
  border-radius: 5px;
  padding: 10px;
  width: unset;
  margin: 15px;
  height: calc(90vh - 50px);
  overflow: auto;
}
.overflow-hide:hover {
  overflow: hidden;
}
.ssbox.hover-new {
  background: #fdfdfd;
  border: 0.3px solid rgba(10, 86, 200, 0.5);
  box-sizing: border-box;
  box-shadow: inset 0px 0px 4px rgba(0, 0, 0, 0.25);
  border-radius: 1px;

  height: 87px;
}

/* tstconfig tabs */

.sidetestconfig .nav-tabs .nav-link.active,
.nav-tabs .nav-item.show .nav-link {
  font-weight: 600;
  font-size: 14px;

  color: #72738e;
}
.sidetestconfig a {
  color: #495057;

  font-weight: 400;
  font-size: 14px;
  line-height: 21px;
  color: #adaaaa;
}
.sidetestconfig .sidedatetime input {
  overflow: visible;
  background: none;
  border: 1px solid var(--input-border);
  border-radius: 3px !important;
  height: calc(1.5em + 0.75rem + 2px);
  padding: 0.375rem 0.75rem;
  font-size: var(--size-12);
  background: var(--bg-inputfield);
  color: #6d6d6d;
  width: 100%;
  box-shadow: inset 0px 0px 4px rgb(0 0 0 / 25%);
}
.calender-image {
  position: absolute;
  top: 3px;
  right: 5px;
}
.sidetestconfig .nav-tabs .nav-link:hover,
.nav-tabs .nav-link:focus {
  border-color: unset;
}
.sidetestconfig .nav-tabs .nav-link {
  margin-bottom: 1px;
  border: unset;
}
.sidetestconfig a:hover {
  color: #495057;
}
.sidetestconfig .nav-tabs {
  border-bottom: 1px solid #dee2e6;
  justify-content: space-between;
}
.sidetestconfig .nav-link {
  display: block;
  padding: 0.5rem 0rem;
}
.sidetestconfig .nav-tabs .nav-link.active {
  position: relative;
}
.sidetestconfig .nav-tabs .nav-link.active::before {
  content: "";
  height: 5px;
  background-color: #72738e;
  width: 103%;
  position: absolute;
  bottom: -1px;
  border-radius: 5px;
}
.sidetestconfig .tab-pane {
  height: 45vh;
}
.disabledfieldsetStyle {
  color: var(--bg-tertiary);
  background-color: #ebebeb;
  position: absolute;
  left: 0.5vw;
  top: -9px;
  font-size: var(--size-10);
  line-height: 18px;
  z-index: 5;
  padding-left: 6px;
  padding-right: 4px;
}
.testconfig-topDiv > .inputField {
  font-family: "Poppins", sans-serif;
  border: 1px solid var(--input-border);
  /* padding: 5px; */
  border-radius: 3px !important;
  font-size: var(--size-12);
  /* opacity: 0.4; */
  background: var(--bg-inputfield);
  color: #6d6d6d;
  box-shadow: inset 0px 0px 4px rgb(0 0 0 / 25%);
  box-sizing: border-box;
}
.testconfig-saveproceed {
  position: absolute;
  bottom: 10px;
}
.body-viewtrue-col {
  width: 100%;
}
.testconfig-tp-text {
  font-weight: var(--weight-M);
  font-size: 11px;
  line-height: 16px;
  color: #696969;
}
.testconfig-pinktext {
  color: var(--bg-secondary);
  font-size: var(--size-12);
  font-weight: var(--weight-LM);
  line-height: 16px;
}
.scroll-flow {
  overflow-y: auto;
  overflow-x: hidden;

  height: calc(100vh - 225px);
  display: block;
  border-radius: 5px;
  margin-top: 2px;
  width: calc(100% - -7px);
}
.schedule-meeting-style {
  font-family: "Poppins";
  font-style: normal;
  font-weight: 500;
  font-size: 12px;

  color: #696969;
}
.kanban .addbtn {
  font-size: 12px;
  padding: 4px 7px;
}
.availability-class {
  font-family: "Poppins";
  font-style: normal;
  font-weight: 400;
  font-size: 12px;

  color: #696969;
}
.jdd-xMarkCancel {
  position: absolute;
  top: -25px;
  right: -25px;
  border-radius: 50px;
  height: 20px;
  width: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #72738e;
  cursor: pointer;
}
</style>
