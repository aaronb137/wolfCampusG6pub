<template>
  <!--Developed by Clint Vega
This component is used to create new posts to the explore page, it takes the post content and category as inputs-->
  <v-form>
    <v-card :class="{ shake: badAlert }">
      <v-alert
        v-model="badAlert"
        type="error"
        title="Error Submitting Post"
        closable
        close-label="Close"
        v-if="err"
        >{{ err }}</v-alert
      >
      <v-card-title class="text-h5">New Post</v-card-title>

      <v-text-field
        v-model="postContent"
        label="New Post"
        variant="outlined"
        style="
          max-width: 95%;
          border: none;
          min-height: 210px;
          margin-bottom: -60px;
        "
      ></v-text-field>

      <v-select
        v-model="selectedCategory"
        :items="categories.map((category) => category.title)"
        label="Select a category"
        style="max-width: 30%; margin-left: 2%; border: none; margin-top: -3px"
      ></v-select>

      <v-row style="justify-content: center">
        <v-col cols="4">
          <v-btn
            class="saveBtn"
            style="margin-left: -4px"
            block
            @click="
              submitPost();
              $emit('cancel-post');
            "
            >Post</v-btn
          >
        </v-col>
        <v-col cols="4">
          <v-btn class="cancelBtn" block @click="$emit('cancel-post')"
            >Cancel</v-btn
          >
        </v-col>
      </v-row>
    </v-card>
  </v-form>
</template>

<script>
import axios from "axios";
import ErrorService from "@/helperfuncts/errorHelper.js";
import { ref } from "vue";
import { db, auth } from "@/firebase";
import {
  collection,
  serverTimestamp,
  onSnapshot,
  getDoc,
  doc,
} from "firebase/firestore";
export default {
  name: "ExplorePost",
  setup() {
    // Create a ref for the categories collection
    const categories = ref([]);
    const categoriesRef = collection(db, "categories");

    // Listen for changes to the categories collection
    //Used to populate the category dropdown that the user will assign their post to.
    onSnapshot(categoriesRef, (querySnapshot) => {
      const newCategories = [];
      querySnapshot.forEach((doc) => {
        newCategories.push({ id: doc.id, ...doc.data() });
      });
      categories.value = newCategories;
    });

    return { categories };
  },
  data() {
    return {
      goodAlert: false,
      badAlert: false,
      postContent: "",
      selectedCategory: "",
      err: "",
    };
  },
  methods: {
    //When user clicks submit, their post information is added to a new
    //document in the posts collection, including the inputted content and category.
    async submitPost() {
      const userRef = doc(db, "users", auth.currentUser.uid);
      const userSnap = await getDoc(userRef);
      this.goodAlert = false;
      this.badAlert = false;

      axios
        .post("http://localhost:5000/process", {
          content: this.postContent,
          category: this.selectedCategory,
          PostDate: serverTimestamp(),
          uid: auth.currentUser.uid,
          firstName: userSnap.data().FirstName,
          lastName: userSnap.data().LastName,
          likes: [],
          isDeleted: false,
        })
        .then((response) => {
          this.goodAlert = true;
          this.response = "Post submitted!";
          console.log("Post saved to database:", response.data);
        })
        .catch((err) => {
          this.badAlert = true;
          let errorMessage = ErrorService.getErrorMessage(err.response.status);
          this.err = errorMessage;
          console.log(errorMessage);
        });

      this.postContent = "";
      this.selectedCategory = "";
    },
  },
};
</script>

<style scoped>
.v-form {
  .shake {
    animation: shake 0.5s;
  }

  @keyframes shake {
    10%,
    90% {
      transform: translate3d(-1px, 0, 0);
    }

    20%,
    80% {
      transform: translate3d(2px, 0, 0);
    }

    30%,
    50%,
    70% {
      transform: translate3d(-4px, 0, 0);
    }

    40%,
    60% {
      transform: translate3d(4px, 0, 0);
    }
  }

  position: absolute;
  z-index: 1;
  margin-left: 25%;
  margin-right: 25%;
  margin-top: 5%;
  margin-bottom: 5%;
  width: 50%;
}

.v-card {
  border-radius: 10px;
  margin: 5%;
  height: 92%;
  box-shadow: 0 16px 56px rgba(0, 0, 0, 0.1), 0 16px 56px rgba(0, 0, 0, 0.1),
    0 16px 56px rgba(0, 0, 0, 0.1);
}

.v-card-title {
  color: black;
  text-align: center;
}

.v-avatar {
  height: 100%;
  width: 100%;
  margin: 5px;
}

.v-btn {
  border-width: 2px;
  margin-left: 11%;
  margin-bottom: 3%;
}

.v-text-field {
  border-color: #4a6fa5;
  border: solid;
  border-width: 2px;
  margin-left: 2%;
}

@media (max-width: 600px) {
  .v-card {
    width: 80%;
    margin: 10% auto;
  }
}

.addClass {
  /* margin-left: 11%; */
  margin-bottom: 3%;
  margin-inline: 40%;
  background-color: #c0d6df;
}

.saveBtn {
  background-color: #56a0ce;
  border: none;
}

@media (max-width: 400px) {
  .v-card {
    width: 90%;
    margin: 20% auto;
  }
}
</style>
