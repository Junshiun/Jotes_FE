<template>
  <actionContainer
    page="Edit"
    :type="1"
    btnOne="delete"
    :note="theNote"
    @buttonOne="deleteNote"
    @buttonConfirm="editHandler"
  ></actionContainer>
  <loadingPage :show="loadShow"></loadingPage>
</template>

<script>
import { ref } from "@vue/reactivity";
import { useStore } from "vuex";
import { onBeforeRouteLeave } from "vue-router";
import { router } from "../../router";
import actionContainer from "./actionContainer.vue";
import loadingPage from "../loadingPage.vue";

export default {
  name: "editPage",
  props: ["note"],
  components: {
    actionContainer,
    loadingPage,
  },
  setup(props) {
    const theNote = ref(props.note);
    const store = useStore();

    let loadShow = ref(false);

    let success = false;
    const editHandler = async ({ title, category, content }) => {
      loadShow.value = true;

      const response = await store.dispatch({
        type: "editNote",
        payload: {
          title: title,
          category: category,
          content: content,
          id: theNote.value._id,
        },
      });

      if (response !== true) console.log(response);
      else {
        success = true;
        store.dispatch({ type: "fetchNotes" });
        router.push({ path: `/notes/${theNote.value._id}` });
      }

      loadShow.value = false;
    };

    const deleteNote = async () => {
      loadShow.value = true;

      let response = await store.dispatch({
        type: "deleteNote",
        payload: { id: theNote.value._id },
      });

      if (response !== true) console.log(response);
      else if (response === true) {
        success = true;
        store.dispatch({ type: "fetchNotes" });
        router.push({ path: "/notes" });
      }

      loadShow.value = false;
    };

    onBeforeRouteLeave((to, from, next) => {
      if (success !== true) {
        const yes = confirm("all changes will not be saved. Are you sure?");

        if (yes) next();
        else next(false);
      } else next();
    });

    return { theNote, editHandler, deleteNote, loadShow };
  },
};
</script>

<style scoped>
.inputTitle_np {
  font-size: 1.5rem;
}

.inputContent_np {
  width: 100%;
  height: 15rem;
}

button[type="submit"] {
  border: none;
  padding: 0.5rem;
  float: right;
  cursor: pointer;
}

button[type="submit"]:hover {
  background-color: #bcbcbc;
}
</style>
