<template>
  <div class="q-pa-md">
    <q-form @submit="search" @reset="onReset" class="q-gutter-md">
      <q-select
        filled
        v-model="selectedLanguages"
        multiple
        :options="languages"
        label="Programming Languages*"
      ></q-select>
      <q-input
        filled
        v-model="inputKeywords"
        hint="Keywords to search for"
        label="Keywords"
      >
      </q-input>
      <q-input
        filled
        v-model="inputCreated"
        label="Create date expression"
        hint="Repo creation date expression to search for"
        :rules="[
          (val) =>
            this.isValidDateExpression(val) ||
            'Please use YYYY-MM-DD or >YYYY-MM-DD format',
        ]"
      ></q-input>
      <q-input
        filled
        type="number"
        v-model="page"
        label="Page number"
        hint="The page number to search for"
        lazy-rules
        :rules="[
          (val) =>
            (val !== null && val !== '') || 'Please enter the page number',
          (val) =>
            (val > 0 && val < 100001) ||
            'Please enther a number between 1 and 100000',
        ]"
      ></q-input>
      <q-input
        filled
        type="number"
        v-model="rowsPerPage"
        label="Repos per page"
        hint="Maximum number of repos per page"
        lazy-rules
        :rules="[
          (val) =>
            (val !== null && val !== '') ||
            'Please enter the maximum number of repos per page',
          (val) =>
            (val > 0 && val < 1001) ||
            'Please enther a number between 1 and 1000',
        ]"
      ></q-input>

      <div>
        <q-btn label="Submit" type="submit" color="primary"></q-btn>
        <q-btn
          label="Reset"
          type="reset"
          color="primary"
          flat
          class="q-ml-sm"
        ></q-btn>
      </div>
    </q-form>

    <q-dialog v-model="showRepoCard">
      <q-card class="my-card">
        <q-card-section>
          <div class="row no-wrap items-center">
            <div class="col text-h6 ellipsis">Repo detailed JSON info</div>
          </div>
          <div>
            {{ detailedInfo }}
          </div>
        </q-card-section>
        <q-separator></q-separator>
      </q-card>
    </q-dialog>
    <q-table
      v-if="hasSearched"
      class="custom-table"
      title="Repositories"
      :data="data"
      :columns="columns"
      row-key="id"
      virtual-scroll
      :pagination.sync="pagination"
      :rows-per-page-option="[0]"
    >
      <template v-slot:top>
        <strong> Total Number of Repositories: {{ totalCount }}</strong>
        <q-space></q-space>
        <img
          style="height: 35px; width: 35px"
          src="https://cdns.iconmonstr.com/wp-content/assets/preview/2012/240/iconmonstr-github-3.png"
        />
      </template>
      <template v-slot:body-cell-action="props">
        <q-td :props="props">
          <q-btn
            icon-right="info"
            color="secondary"
            no-caps
            flat
            dense
            @click="actionEval(data.indexOf(props.row))"
          />
        </q-td>
      </template>
      <template v-slot:body-cell-htmlurl="props">
        <q-td :props="props">
          <q-btn
            icon-right="home"
            no-caps
            flat
            color="secondary"
            @click="openInNewTab(data[data.indexOf(props.row)].html_url)"
          />
        </q-td>
      </template>
    </q-table>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "Search",
  props: {
    baseEndpoint: String,
  },
  methods: {
    openInNewTab(url) {
      if (url !== undefined) {
        var win = window.open(url, "_blank");
        win.focus();
      }
    },
    actionEval(index) {
      this.detailedInfo = JSON.stringify(
        JSON.parse(JSON.stringify(this.data[index])),
        null,
        2
      );
      this.showRepoCard = true;
    },
    sleep(ms) {
      return new Promise((resolve) => setTimeout(resolve, ms));
    },
    isValidDate(dateExpr) {
      return (
        new Date(dateExpr) !== "Invalid Date" && !isNaN(new Date(dateExpr))
      );
    },
    isValidDateExpression(dateExpr) {
      return this.isValidDate(dateExpr.replace(">", ""));
    },
    async search() {
      this.data = [];
      this.hasSearched = true;
      const maxRetries = 3;

      for (let retry = 0; retry < maxRetries; retry++) {
        try {
          const res = await axios.post(
            `${this.baseEndpoint}/repos/search?per_page=${this.rowsPerPage}&page=${this.page}`,
            this.getSearchPayload(
              this.selectedLanguages,
              this.inputCreated,
              this.inputKeywords
            ),
            this.getHeaders()
          );
          this.totalCount = res.data.pagination.total_count;
          this.pagination.page = 1;
          this.data.splice(0, res.data.items.length, ...res.data.items);
          return;
        } catch (err) {
          if (
            retry == maxRetries - 1 ||
            (err.response !== undefined && err.response.status < 500)
          ) {
            let error_msg = "Failed to submit: ";
            if (err.response === undefined) {
              error_msg += " Network Failure";
            } else {
              error_msg += JSON.stringify(err.response.data);
            }
            this.$q.notify({
              color: "red-5",
              textColor: "white",
              message: error_msg,
              position: "center",
            });
            return;
          }
          await this.sleep(1000);
        }
      }
    },
    getHeaders() {
      return {
        headers: {
          Accept: "application/json",
        },
      };
    },
    getSearchPayload(languages, created, keywords) {
      var quals = [];
      var words = [];
      for (var i = 0; i < languages.length; i++) {
        quals.push({ language: languages[i] });
      }
      if (created !== undefined) {
        quals.push({ created: created });
      }
      if (keywords !== undefined) {
        words = keywords.split(" ");
      }
      return {
        qualifiers: quals,
        keywords: words,
      };
    },
    onReset() {
      this.hasSearched = false;
      this.selectedLanguages = ["Elixir"];
      this.rowsPerPage = 5;
      this.page = 1;
      this.inputCreated = ">2020-01-01";
    },
  },
  computed: {
    isPerPageValid() {
      if (this.rowsPerPage === null || this.rowsPerPage === undefined) {
        return false;
      }
      return this.rowsPerPage > 0 && this.rowsPerPage < 1001;
    },
  },
  data: () => {
    return {
      showRepoCard: false,
      detailedInfo: "",
      hasSearched: false,
      submitted: false,
      page: 1,
      rowsPerPage: 5,
      totalCount: 0,
      selectedLanguages: ["Elixir"],
      inputKeywords: undefined,
      // inputCreated could have a better UX than a raw string expression
      inputCreated: ">2020-01-01",
      languages: [
        "C",
        "D",
        "Elixir",
        "Erlang",
        "Elm",
        "Java",
        "JavaScript",
        "Python",
        "TypeScript",
        "Ruby",
        "Rust",
        "Scala",
      ],
      pagination: {
        page: 1,
        rowsPerPage: 5,
      },
      columns: [
        {
          name: "desc",
          required: true,
          label: "Name",
          align: "left",
          field: (row) => row.name,
          format: (val) => `${val}`,
          sortable: true,
        },
        {
          name: "fullname",
          align: "left",
          label: "Full name",
          field: (row) => row.full_name,
          sortable: true,
        },
        {
          name: "language",
          align: "left",
          label: "Language",
          field: (row) => row.language,
          sortable: true,
        },
        {
          name: "stars",
          align: "left",
          label: "Stars",
          field: (row) => row.stargazers_count,
          sortable: true,
        },
        {
          name: "created_at",
          align: "left",
          label: "Created at",
          field: (row) => row.gh_created_at,
          sortable: true,
        },
        {
          name: "htmlurl",
          label: "GitHub URL",
          align: "left",
          field: "html_url",
        },
        {
          name: "action",
          label: "More details",
          align: "left",
          field: "action",
        },
      ],
      data: [],
    };
  },
};
</script>

<style lang="sass">
.custom-table
  height: 420px

  .q-table__top,
  .q-table__bottom,
  thead tr:first-child th
    background-color: #f2f2f2

  thead tr th
    position: sticky
    z-index: 1
  thead tr:first-child th
    top: 0

  &.q-table--loading thead tr:last-child th
    top: 48px
</style>
