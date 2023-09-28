<template>
  <div>
    <!-- Import and display DashboardFriends component -->
    <DashboardFriends />
    <!-- Import and display AddClass component with the closeModal event handler and clickOutside directive -->
    <AddClass class="addclass" v-if="showModal" @close-modal="closeModal" v-click-outside="closeModal"></AddClass>
    <!-- Import and display MyClasses component with the add-class event handler -->
    <MyClasses @add-class="openModal"></MyClasses>
    <!-- Import and display ProfileAvatar component -->
    <ProfileAvatar />
    <WallPost />
    <!-- Import and display WallPost component -->
    <WolfFeed @post-selected="selectedPost" @post-to-delete="deletedPost" />
    <ModifyModal
      v-if="showModify"
      v-click-outside="{
        handler: closeModify,
        include: includeFunction,
      }"
      :selectedPost="post"
      @postHandler="handlePost"
      @good-post="closeModify"
      >{{ logModal }}</ModifyModal
    >

    <DeleteModal
      v-if="showDelete"
      v-click-outside="{
        handler: closeDelete,
        include: includeFunction,
      }"
      :deletedPost="post"
      @deleteHandler="handleDelete"
    />
    <!-- Import and display NavBar component -->
    <nav-bar></nav-bar>
    <!-- Import and display WolfFeed component -->
  </div>
</template>

<script>
import DashboardFriends from "@/components/Dashboard/DashboardFriends.vue";
import MyClasses from "@/components/Dashboard/MyClasses.vue";
import ProfileAvatar from "@/components/Dashboard/ProfileAvatar.vue";
import WallPost from "@/components/Dashboard/WallPost.vue";
import WolfFeed from "@/components/Dashboard/WolfFeed.vue";
import AddClass from "@/components/Dashboard/AddClass.vue";
import ModifyModal from "@/components/Dashboard/ModifyPost.vue";
import DeleteModal from "@/components/Dashboard/DeletePost.vue"

export default {
  name: "HomeView",
  components: {
    DashboardFriends,
    MyClasses,
    ProfileAvatar,
    WallPost,
    WolfFeed,
    AddClass,
    ModifyModal,
    DeleteModal
  },
  data() {
    return {
      showModal: false,
      showModify: false,
      showDelete: false,
      badPost: false,
      post: null,
    };
  },
  directives: {
    clickOutside: {
      // Custom directive to handle clicks outside the target element
      bind(el, binding, vnode) {
        el.clickOutsideEvent = function(event) {
          if (!(el == event.target || el.contains(event.target))) {
            vnode.context[binding.expression](event);
          }
        };
        document.body.addEventListener("click", el.clickOutsideEvent);
      },
      unbind(el) {
        document.body.removeEventListener("click", el.clickOutsideEvent);
      },
    },
  },
  methods: {
    logModal() {
      console.log(this.showModify);
    },
    handleDelete(success) {
      if (success) {
        this.showDelete = false;
      } else {
        this.openModify();
      }
    },
    handlePost(success) {
      if (success) {
        this.showModify = true;
      } else {
        this.closeModify();
      }
    },
    openModal() {
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
    },
    openModify() {
      this.showModify = true;
    },
    closeModify() {
      this.showModify = false;
      console.log("IN CLOSE MODIFY:");
      console.log(this.showModify);
    },
    openDelete() {
      this.showDelete = true;
    },
    closeDelete() {
      this.showDelete = false;
    },
    selectedPost(post) {
      this.post = post;
      this.openModify();
      console.log("IN SELECTEDPOST:");
      console.log(this.showModify);
    },
    deletedPost(post) {
      this.post = post;
      this.openDelete();
    },
    includeFunction() {
      //visible elements
      const tempArray = [document.querySelector(".included")];
      if (document.querySelector(".v-dialog")) {
        //push the element to array when it is visible
        tempArray.push(document.querySelector(".v-dialog"));
      }
      return tempArray;
    },
  },
};
</script>


<style scoped>
body {
  background-color: #fdf0d5;
}


</style>
