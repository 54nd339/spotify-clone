<script setup>
import { ref, onMounted, computed } from "vue";
import MenuItem from "./components/MenuItem.vue";
import MusicPlayer from "./components/MusicPlayer.vue";
import ChevronUp from "vue-material-design-icons/ChevronUp.vue";
import ChevronDown from "vue-material-design-icons/ChevronDown.vue";
import LoginModal from "./components/LoginModal.vue";
import ConfirmModal from "./components/ConfirmModal.vue";
import { auth } from "@/db/config";
import useAuth from "@/db/useAuth";

import { useSongStore, useVideoStore } from "./stores";
import { storeToRefs } from "pinia";

const useSong = useSongStore();
const { isAudioPlaying, currentAudioTrack } = storeToRefs(useSong);
const useVideo = useVideoStore();
const { isVideoPlaying, currentVideoTrack } = storeToRefs(useVideo);

const user = ref(auth.currentUser);
const username = ref("");
const uid = ref("");

auth.onAuthStateChanged((param) => {
  user.value = param;
  if (param) {
    username.value = param.displayName;
    uid.value = param.uid;
  } else {
    username.value = "User";
    uid.value = "";
  }
});

const getLogoImage = () => {
  return new URL(window.innerWidth > 768
    ? "/images/icons/spotify-logo.png"
    : "/images/icons/spotify-logo.png", import.meta.url).href;
};

const logoImage = ref(getLogoImage());
onMounted(() => {
  isAudioPlaying.value = false;
  window.addEventListener("resize", () => {
    logoImage.value = getLogoImage();
  });
});

const openMenu = ref(false);
const showModal = ref(false);

const userDetail = (user) => {
  showModal.value = false;
  if (user) {
    username.value = user.displayName;
    uid.value = user.uid;
  }
};

const greeting = ref("Hello");
const date = new Date();
const hour = date.getHours();
if (hour < 12) {
  greeting.value = "Good Morning";
} else if (hour < 18) {
  greeting.value = "Good Afternoon";
} else {
  greeting.value = "Good Evening";
}

const showConfirmModal = ref(false);
const closeConfirm = () => {
  openMenu.value = false;
  showConfirmModal.value = false;
};
const logout = async () => {
  openMenu.value = false;
  await useAuth().logout().then(() => {
    username.value = "User";
    showConfirmModal.value = false;
  });
}

</script>

<template>
  <div>
    <div
      id="TopNav"
      class="w-[calc(100%-70px)] md:w-[calc(100%-240px)] h-[60px] fixed right-0 z-20 bg-[#101010] bg-opacity-80 flex items-center justify-between"
    >
      <div class="flex items-center ml-6">
        <h1 class="text-white text-[15px] md:text-[30px] mr-4 font-semibold">
          {{ greeting }}, {{ username }}
        </h1>
      </div>

      <button
        id="loginbtn"
        v-if="!user"
        @click="showModal = true"
        class="bg-black hover:bg-[#1DB954] rounded-full p-0.5 mr-8 mt-0.5 cursor-pointer"
      >
        <div class="flex items-center">
          <div class="text-white text-[14px] mx-1 font-semibold">
            Login/Register
          </div>
        </div>
      </button>
      <button
        v-else
        @click="openMenu = !openMenu"
        :class="openMenu ? 'bg-[#282828]' : 'bg-black'"
        class="bg-black hover:bg-[#282828] rounded-full p-0.5 mr-8 mt-0.5 cursor-pointer"
      >
        <div class="flex items-center">
          <img
            class="rounded-full"
            width="27"
            src="https://www.okshop.in/static/assets/user_placeholder_circle.ecad1f96.png"
          />
          <div class="text-white text-[14px] ml-1.5 font-semibold">
            {{ username }}
          </div>
          <ChevronDown
            v-if="!openMenu"
            @click="openMenu = true"
            fillColor="#FFFFFF"
            :size="25"
          />
          <ChevronUp
            v-else
            @click="openMenu = false"
            fillColor="#FFFFFF"
            :size="25"
          />
        </div>
      </button>

      <Suspense>
      <span
        v-if="openMenu"
        class="fixed w-[190px] bg-[#282828] shadow-2xl z-50 rounded-sm top-[52px] right-[35px] p-1 cursor-pointer"
      >
        <ul class="text-gray-200 font-semibold text-[14px]">
          <RouterLink to="/profile">
            <li
              class="px-3 py-2.5 hover:bg-[#3E3D3D] border-b border-b-gray-600"
              @click="openMenu = false"
            >
              Profile
            </li>
          </RouterLink>
          <li
            class="px-3 py-2.5 hover:bg-[#3E3D3D]"
            @click="showConfirmModal = true"
          >
            Logout
          </li>
        </ul>
      </span>
      </Suspense>
    </div>

    <div
      id="SideNav"
      class="h-[100%] p-6 md:w-[240px] w-[70px] fixed z-50 bg-black"
    >
      <RouterLink to="/">
        <img class="md:w-[125px] md:flex" :src="logoImage" />
      </RouterLink>
      <div class="my-8"></div>
      <Suspense>
      <ul>
        <RouterLink to="/">
          <MenuItem
            class="ml-[1px]"
            :iconSize="23"
            name="Home"
            iconString="home"
            pageUrl="/"
          />
        </RouterLink>
        <RouterLink to="/search">
          <MenuItem
            class="ml-[1px]"
            :iconSize="24"
            name="Search"
            iconString="search"
            pageUrl="/search"
          />
        </RouterLink>
        <RouterLink to="/library">
          <MenuItem
            class="ml-[2px]"
            :iconSize="23"
            name="Your Library"
            iconString="library"
            pageUrl="/library"
          />
        </RouterLink>
        <div class="py-3.5"></div>
        <MenuItem
          :iconSize="24"
          name="Create Playlist"
          iconString="playlist"
          pageUrl="/playlist"
        />
        <MenuItem
          class="-ml-[1px]"
          :iconSize="27"
          name="Liked Songs"
          iconString="liked"
          pageUrl="/liked"
        />
      </ul>
      </Suspense>
      <div class="border-b border-b-gray-700"></div>
    </div>
  </div>

  <LoginModal v-if="showModal" @close="userDetail" />
  <ConfirmModal
    v-if="showConfirmModal"
    @close="closeConfirm"
    @confirm="logout"
  />

  <Suspense>
  <div
    class="fixed right-0 top-0 w-[calc(100%-70px)] md:w-[calc(100%-240px)] overflow-auto h-full bg-gradient-to-b from-[#1C1C1C] to-black"
  >
    <div class="mt-[70px]"></div>
    <RouterView />
    <div class="mb-[100px]"></div>
  </div>
  </Suspense>

  <MusicPlayer v-if="currentAudioTrack" />
</template>
