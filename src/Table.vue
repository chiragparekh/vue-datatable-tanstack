<template>
  <div class="-mx-4 -my-2 overflow-x-auto sm:-mx-6 lg:-mx-8">
    <div v-if="loading">Loading data...</div>

    <div class="inline-block min-w-full py-2 align-middle sm:px-6 lg:px-8">
      <div
        class="overflow-hidden shadow ring-1 ring-black ring-opacity-5 sm:rounded-lg"
      >
        <table class="min-w-full divide-y divide-gray-300">
          <thead class="bg-gray-50">
            <tr
              v-for="headerGroup in table.getHeaderGroups()"
              :key="headerGroup.id"
            >
              <th
                v-for="header in headerGroup.headers"
                :key="header.id"
                :colspan="header.colSpan"
                class="py-3.5 pl-4 pr-3 text-left text-sm font-semibold text-gray-900 sm:pl-6"
                :style="{ width: header.getSize() + '%' }"
              >
                <FlexRender
                  :render="header.column.columnDef.header"
                  :props="header.getContext()"
                ></FlexRender>
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-200 bg-white">
            <tr
              v-for="row in table.getRowModel().rows"
              :key="row.id"
              class="hover:bg-gray-100"
            >
              <td
                v-for="cell in row.getVisibleCells()"
                :key="cell.id"
                class="whitespace-nowrap py-2 pl-4 pr-3 text-sm font-medium text-gray-900 sm:pl-6"
              >
                <FlexRender
                  :render="cell.column.columnDef.cell"
                  :props="cell.getContext()"
                ></FlexRender>
              </td>
            </tr>
          </tbody>
        </table>
        <div
          class="flex items-center justify-between border-t border-gray-200 bg-white px-4 py-3 sm:px-6"
        >
          <div class="flex flex-1 justify-between sm:hidden">
            <a
              href="#"
              class="relative inline-flex items-center rounded-md border border-gray-300 bg-white px-4 py-2 text-sm font-medium text-gray-700 hover:bg-gray-50"
              >Previous</a
            >
            <a
              href="#"
              class="relative ml-3 inline-flex items-center rounded-md border border-gray-300 bg-white px-4 py-2 text-sm font-medium text-gray-700 hover:bg-gray-50"
              >Next</a
            >
          </div>
          <div
            class="hidden sm:flex sm:flex-1 sm:items-center sm:justify-between"
          >
            <div>
              <p class="text-sm text-gray-700">
                Showing
                {{ " " }}
                <span class="font-medium">{{ data.skip + 1 }}</span>
                {{ " " }}
                to
                {{ " " }}
                <span class="font-medium">{{ data.skip + data.limit }}</span>
                {{ " " }}
                of
                {{ " " }}
                <span class="font-medium">{{ data.total }}</span>
                {{ " " }}
                results
              </p>
            </div>
            <div>
              <nav
                class="isolate inline-flex -space-x-px rounded-md shadow-sm"
                aria-label="Pagination"
              >
                <button
                  href="#"
                  class="relative inline-flex items-center rounded-l-md px-2 py-2 text-gray-400 ring-1 ring-inset ring-gray-300 hover:bg-gray-50 focus:z-20 focus:outline-offset-0"
                  @click.prevent="table.previousPage()"
                  :disabled="!table.getCanPreviousPage()"
                >
                  <span class="sr-only">Previous</span>
                  <ChevronLeftIcon class="h-5 w-5" aria-hidden="true" />
                </button>
                <template v-if="table.getPageCount() >= 0">
                  <template
                    v-for="i in table.getPageCount()"
                    :key="`page-${i}`"
                  >
                    <button
                      aria-current="page"
                      class="relative z-10 inline-flex items-center bg-indigo-600 px-4 py-2 text-sm font-semibold text-white focus:z-20 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
                      v-if="currentPage == i"
                    >
                      {{ i }}
                    </button>
                    <button
                      aria-current="page"
                      class="relative hidden items-center px-4 py-2 text-sm font-semibold text-gray-900 ring-1 ring-inset ring-gray-300 hover:bg-gray-50 focus:z-20 focus:outline-offset-0 md:inline-flex"
                      @click="changePage(i)"
                      v-else
                    >
                      {{ i }}
                    </button>
                  </template>
                </template>
                <button
                  href="#"
                  class="relative inline-flex items-center rounded-r-md px-2 py-2 text-gray-400 ring-1 ring-inset ring-gray-300 hover:bg-gray-50 focus:z-20 focus:outline-offset-0"
                  @click.prevent="table.nextPage()"
                  :disabled="!table.getCanNextPage()"
                >
                  <span class="sr-only">Next</span>
                  <ChevronRightIcon class="h-5 w-5" aria-hidden="true" />
                </button>
              </nav>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="jsx" setup>
import {
  TrashIcon,
  ChevronLeftIcon,
  ChevronRightIcon,
} from "@heroicons/vue/24/outline";

import {
  FlexRender,
  getCoreRowModel,
  useVueTable,
  createColumnHelper,
  getPaginationRowModel,
} from "@tanstack/vue-table";
import { computed, onMounted, reactive, ref, watch } from "vue";

const columnHelper = createColumnHelper();

const removeProduct = (row) => {};

const columns = [
  columnHelper.accessor("id", {
    header: "ID",
    size: 5,
  }),
  columnHelper.accessor("title", {
    header: "Title",
    size: 30,
  }),
  columnHelper.accessor("brand", {
    header: "Brand",
  }),
  columnHelper.accessor("price", {
    header: "Price",
    size: 15,
  }),
  columnHelper.accessor("stock", {
    header: "Stock",
    size: 15,
  }),
  columnHelper.display({
    id: "actions",
    size: 10,

    cell: (props) => (
      <div className="flex space-x-2 justify-end">
        <button
          type="button"
          className="inline-flex items-center gap-x-1.5 border border-red-700 rounded-md text-red-700 px-2.5 py-1.5 text-sm font-semibold hover:text-red-900 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-red-600"
          onClick={removeProduct(props.row)}
        >
          <TrashIcon class="h-5 w-5" aria-hidden="true" />
        </button>
      </div>
    ),
  }),
];

const data = reactive({
  limit: 15,
  products: [],
  skip: 0,
  total: 0,
});

const currentPage = computed(() => data.skip / data.limit + 1);

const totalPages = computed(() =>
  data.total ? Math.ceil(data.total / data.limit) : -1
);

const loading = ref(false);

const loadData = () => {
  loading.value = true;
  fetch(`https://dummyjson.com/products?skip=${data.skip}&limit=${data.limit}`)
    .then((res) => res.json())
    .then((json) => {
      data.products = json.products;
      data.skip = json.skip;
      data.total = json.total;
      loading.value = false;
      table.setPageCount(10);
    });
};
onMounted(() => {
  loadData();
});

const setPagination = (updater) => {
  updater({
    pageIndex: 5,
    pageSize: data.limit,
    pageCount: 100,
  });
};

const paginationState = reactive({
  pageSize: data.limit,
  pageIndex: 1,
});

const pageCount = ref(-1);

watch(pageCount, (newPageCount, oldPageCount) => {
  table.setPageCount(newPageCount);
});

const table = useVueTable({
  defaultColumn: {
    size: "auto",
    minSize: 5,
    maxSize: Number.MAX_SAFE_INTEGER,
  },
  columns: columns,
  get data() {
    return data.products;
  },
  getCoreRowModel: getCoreRowModel(),
  debugTable: true,
  manualPagination: true,
  state: {
    pagination: paginationState,
  },
  onPaginationChange: setPagination,
});

const changePage = (page) => {
  console.log(page);
  // data.skip = data.limit * page - data.limit;
  // table.setPageIndex(page);
  // paginationState.pageIndex = page;

  // loadData();
};
</script>
