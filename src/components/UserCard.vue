<template>
  <div class="UserCard container-xl row justify-content-start">
    <div
      height="100%"
      class="pfp col-2"
      :style="{ backgroundImage: `url(${user.avatar_url})` }"
    />
    <div class="profile-info col-8">
      <span class="h3 mx-1">{{ user.login }}</span>
      <span class="h4 mx-1 text-muted" v-if="user.type == 'User' && user.name">
        {{ user.name }}
      </span>
      <p class="mt-3">{{ user.bio }}</p>
      <div class="user-repo-count d-flex flex-row justify-content-center">
        <!-- * From F12 in https://github.com/ -->
        <svg
          aria-label="Repository"
          role="img"
          height="16"
          viewBox="0 0 16 16"
          version="1.1"
          width="16"
          data-view-component="true"
          class="octicon octicon-repo"
        >
          <path
            fill-rule="evenodd"
            d="M2 2.5A2.5 2.5 0 014.5 0h8.75a.75.75 0 01.75.75v12.5a.75.75 0 01-.75.75h-2.5a.75.75 0 110-1.5h1.75v-2h-8a1 1 0 00-.714 1.7.75.75 0 01-1.072 1.05A2.495 2.495 0 012 11.5v-9zm10.5-1V9h-8c-.356 0-.694.074-1 .208V2.5a1 1 0 011-1h8zM5 12.25v3.25a.25.25 0 00.4.2l1.45-1.087a.25.25 0 01.3 0L8.6 15.7a.25.25 0 00.4-.2v-3.25a.25.25 0 00-.25-.25h-3.5a.25.25 0 00-.25.25z"
          />
        </svg>
        <p>
          {{ user.public_repos }} Public Repositories ({{ fork_count }} Forks,
          {{ archived_count }} Archived)
        </p>
      </div>
      <p v-if="user.type == 'User'">
        {{ user.followers }} Followers {{ user.following }} Following
      </p>

      <div class="d-flex flex-row justify-content-center mx-auto">
        <button class="btn btn-primary mx-2" @click="openUrl(user.html_url)">
          Visit User Profile
        </button>
        <button
          class="btn btn-outline-primary mx-2"
          @click="openUrl(`${user.html_url}?tab=repositories`)"
        >
          Visit User Repositories
        </button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";
import { User } from "@/backend/types";

@Options({
  props: {
    user: Object,
    fork_count: Number,
    archived_count: Number,
  },
})
export default class UserCard extends Vue {
  user!: User;
  fork_count!: number;
  archived_count!: number;

  openUrl(url: string): void {
    window.open(url, "_blank");
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.pfp {
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  border: 0.4em solid gold;
}
</style>
