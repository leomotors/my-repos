<template>
  <h1 class="display-3 fw-bold mt-5 mb-lg-10">My Repositories</h1>
  <UserCard
    class="mx-auto mt-5"
    :user="user_data"
    :fork_count="fork_count"
    :archived_count="archived_count"
  />
  <hr class="mt-5 mb-4" />
  <div class="input-group btn-toolbar mx-auto mb-4 justify-content-center">
    <div class="input-group-text" id="btnGroupAddon">Sort By</div>
    <button
      type="button"
      v-for="method in sortMethods"
      :key="method.name"
      class="btn btn-outline-secondary"
      :class="{ active: method.name == currentSortMethods }"
      @click="setSortMethod(method)"
    >
      {{ method.name }}
    </button>
  </div>

  <div id="Cards" class="container-fluid row center mx-auto">
    <RepoCard
      class="col-md-6 col-lg-4 col-xl-3"
      v-for="(repo, index) in repos_data"
      :key="repo.name"
      :repo="repo"
      :index="index + 1"
    />
  </div>
  <hr />
  <form id="FooterBar" class="container-md justify-content-center mb-3">
    <label for="name"> Search Other User </label>
    <input
      class="col-sm mx-2"
      type="text"
      id="SearchUser"
      placeholder="Leomotors"
      v-model.trim="user_to_view"
    />
    <button
      class="col-sm mx-1 btn-success btn-sm"
      type="button"
      id="SearchBtn"
      @click.prevent="viewUser"
    >
      View User
    </button>
  </form>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";
import UserCard from "@/components/UserCard.vue";
import RepoCard from "@/components/RepoCard.vue";

import { DefaultUser } from "@/backend/User";
import { User, Repo } from "@/backend/types";

const default_user = "Leomotors";

// * Sort from most to least
function cmp<T extends number | string>(a: T, b: T): number {
  if (a > b) return -1;
  else if (a < b) return 1;
  else return 0;
}

type cmpFunc = (a: Repo, b: Repo) => number;
type sortMethod = { name: string; func: cmpFunc };

const sortMethods: { [type: string]: sortMethod } = {
  recent_updated: {
    name: "Last Pushed",
    func: (a, b) => cmp(a.pushed_at ?? 0, b.pushed_at ?? 0),
  },
  recent_created: {
    name: "Last Created",
    func: (a, b) => cmp(a.created_at ?? 0, b.created_at ?? 0),
  },
  most_stars: {
    name: "Most Stars",
    func: (a, b) => cmp(a.stargazers_count ?? 0, b.stargazers_count ?? 0),
  },
  name: {
    name: "Repo Name",
    func: (a, b) => cmp(b.name.toLowerCase(), a.name.toLowerCase()),
  },
  language: {
    name: "Language",
    func: (a, b) => cmp(b.language ?? "Markdown", a.language ?? "Markdown"),
  },
};

async function loadData(user: string): Promise<{
  user_data: User;
  repos_data: Repo[];
  fork_count: number;
  archived_count: number;
}> {
  let repos_data = [];
  let fork_count = 0;
  let archived_count = 0;

  console.log(`Querying User Data of ${user}`);
  const ures = await fetch(`https://api.github.com/users/${user}`);
  const utxt = await ures.text();
  const user_data = JSON.parse(utxt) as User;

  // * Do not fetch more than 10 times, meaning more than 1k Repo is not supported
  const PAGE_HARD_LIMIT = 10;
  let page = 1;

  while (page <= PAGE_HARD_LIMIT) {
    console.log(`Querying Repos Page #${page}`);
    const rres = await fetch(
      `https://api.github.com/users/${user}/repos?per_page=100&page=${page}`
    );
    page++;
    const rtxt = await rres.text();
    const robj = JSON.parse(rtxt);

    for (const repo of robj) {
      if ((repo as Repo).fork) fork_count++;
      if ((repo as Repo).archived) archived_count++;
      repos_data.push(repo as Repo);
    }

    if (robj.length < 100) break;
  }

  repos_data.sort(sortMethods.recent_updated.func);

  console.log(`Successfully fetched all data of ${user}`);
  return { user_data, repos_data, fork_count, archived_count };
}

@Options({
  components: {
    UserCard,
    RepoCard,
  },
})
export default class App extends Vue {
  user_to_view = "";
  user_data: User = DefaultUser;
  repos_data: Repo[] = [];
  sortMethods = sortMethods;
  currentSortMethods = "Last Pushed";
  fork_count = 0;
  archived_count = 0;

  async created(): Promise<void> {
    const uri = window.location.search.substring(1);
    const params = new URLSearchParams(uri);
    const target_user = params.get("user") || default_user;

    const { user_data, repos_data, fork_count, archived_count } =
      await loadData(target_user);
    this.user_data = user_data;
    this.repos_data = repos_data;
    this.fork_count = fork_count;
    this.archived_count = archived_count;
  }

  setSortMethod(method: sortMethod): void {
    this.currentSortMethods = method.name;
    this.repos_data.sort(method.func);
  }

  viewUser(): void {
    window.location.href =
      window.location.href.split("?")[0] + `?user=${this.user_to_view}`;
  }
}
</script>

<style lang="scss">
#app {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
    Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", Avenir, Helvetica, Arial,
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
