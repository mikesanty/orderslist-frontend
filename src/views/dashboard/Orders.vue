<template>
  <v-container
    id="orders"
    fluid
    tag="section"
  >
    <v-row>
      <v-col
        cols="12"
      >
        <v-card>
          <v-card-title>
            {{ pageListTitle }}
            <v-spacer />
            <v-text-field
              v-model="search"
              append-icon="mdi-magnify"
              label="Search"
              single-line
              hide-details
              class="pt-0 mt-0"
            />
            <v-spacer />
            <v-btn
              small
              depressed
              color="success"
            >
              EXPORT
              <v-btn
                icon
                color="white"
                class="p-0"
                @click="exportExcel"
              >
                <v-icon>mdi-download</v-icon>
              </v-btn>
            </v-btn>
          </v-card-title>
          <br>
          <v-card-text>
            <v-data-table
              :headers="headers"
              :items="orderListData"
              :search.sync="search"
              :options.sync="options"
              :loading="isLoadingorderListData"
              :server-items-length="totalOrdersList"
              loading-text="Loading Data..."
            >
              <template v-slot:item="row">
                <tr>
                  <td class="body-2">
                    {{ row.item.order_name }}
                  </td>
                  <td class="body-2">
                    {{ row.item.cutomer_company }}
                  </td>
                  <td class="body-2">
                    {{ row.item.cutomer_name }}
                  </td>
                  <td class="body-2">
                    {{ row.item.created_at | moment('timezone', 'Australia/Melbourne', 'MMM DD, YYYY hh:mm A') }}
                  </td>
                  <td class="body-2">
                    {{ row.item.delivered_amount }}
                  </td>
                  <td class="body-2">
                    {{ row.item.total_amount }}
                  </td>
                </tr>
              </template>
            </v-data-table>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import axios from 'axios'
  import { json2excel } from 'js2excel'

  export default {
    name: 'Orders',
    data: () => ({
      totalOrdersList: 0,
      search: '',
      headers: [
        {
          text: 'Order Name',
          value: 'order_name',
          class: 'body-1 font-weight-black',
        },
        {
          text: 'Cutomer Company',
          value: 'cutomer_company',
          class: 'body-1 font-weight-black',
        },
        {
          text: 'Cutomer Name',
          value: 'cutomer_name',
          class: 'body-1 font-weight-black',
        },
        {
          text: 'Order Date',
          value: 'created_at',
          class: 'body-1 font-weight-black',
        },
        {
          text: 'Delivered Amount',
          value: 'delivered_amount',
          class: 'body-1 font-weight-black',
        },
        {
          text: 'Total Amount',
          value: 'total_amount',
          class: 'body-1 font-weight-black',
        },
      ],
      isLoadingorderListData: true,
      pageListTitle: 'Order List',
      orderListData: [],
      options: {},
    }),
    computed: {
    },
    watch: {
      options: {
        handler () {
          this.getOrdersList()
        },
      },
      search: {
        handler () {
          this.getOrdersList()
        },
      },
    },
    mounted: function () {
      this.getOrdersList()
    },
    methods: {
      getOrdersList: function () {
        var self = this.$data
        console.log(self.options)
        axios({
          method: 'get',
          url: process.env.VUE_APP_DATA_API_URL,
          params: {
            search: self.search,
            page: self.options.page,
            itemsPerPage: self.options.itemsPerPage,
            sortBy: (self.options.sortBy[0] === undefined ? 'order_date' : self.options.sortBy[0]),
            sortDesc: (self.options.sortDesc[0] === true ? '1' : '-1'),
          },
        }).then(function (response) {
          // console.log(response.data)
          self.totalOrdersList = response.data.pagination.total_records
          self.orderListData = response.data.data.items
          self.isLoadingorderListData = false
        }, (error) => {
          console.log(error)
        })
      },
      exportExcel: function () {
        var self = this.$data
        var data = self.orderListData
        try {
          json2excel({
            data,
            name: self.pageListTitle,
            formateDate: 'MMM DD, YYYY hh:mm A',
          })
        } catch (e) {
          alert('Export Error')
        }
      },
    },
  }
</script>
