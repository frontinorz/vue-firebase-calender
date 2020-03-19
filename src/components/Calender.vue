<template>
  <v-row class="fill-height">
    <v-col>
      <v-sheet height="64">
        <v-toolbar
          flat
          color="white"
        >
          <v-btn
            outlined
            class="mr-4"
            color="grey darken-2"
            @click="setToday"
          >
            今日
          </v-btn>
          <v-btn
            fab
            text
            small
            color="grey darken-2"
            @click="prev"
          >
            <v-icon small>mdi-chevron-left</v-icon>
          </v-btn>
          <v-btn
            fab
            text
            small
            color="grey darken-2"
            @click="next"
          >
            <v-icon small>mdi-chevron-right</v-icon>
          </v-btn>
          <v-toolbar-title>{{ title }}</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-dialog
            v-model="dialog"
            max-width="600px"
          >
            <template v-slot:activator="{ on }">
              <v-btn
                color="primary"
                dark
                v-on="on"
                class="mr-4"
              >新增事件</v-btn>
            </template>
            <v-card :loading="loading">
              <v-card-title>
                <span class="headline">新增事件</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12">
                      <v-text-field
                        label="事件*"
                        required
                        v-model="newEvent.name"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        label="描述"
                        v-model="newEvent.details"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-dialog
                        ref="dialog"
                        v-model="datePickerStart"
                        width="300px"
                      >
                        <template v-slot:activator="{ on }">
                          <v-text-field
                            v-model="newEvent.start"
                            label="開始日期*"
                            readonly
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-date-picker
                          v-model="newEvent.start"
                          scrollable
                          locale="zh-tw"
                          @input="datePickerStart = false"
                        >
                        </v-date-picker>
                      </v-dialog>
                    </v-col>
                    <v-col cols="12">
                      <v-dialog
                        ref="dialog"
                        v-model="datePickerEnd"
                        width="300px"
                      >
                        <template v-slot:activator="{ on }">
                          <v-text-field
                            v-model="newEvent.end"
                            label="結束日期*"
                            readonly
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-date-picker
                          v-model="newEvent.end"
                          scrollable
                          locale="zh-tw"
                          :min="newEvent.start"
                          @input="datePickerEnd = false"
                        >
                        </v-date-picker>
                      </v-dialog>
                    </v-col>
                    <v-col cols="12">
                      <v-dialog
                        ref="dialog"
                        v-model="colorDialog"
                        width="300px"
                      >
                        <template v-slot:activator="{ on }">
                          <v-text-field
                            :background-color="newEvent.color"
                            label="標籤顏色*"
                            value=" "
                            readonly
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-color-picker v-model="newEvent.color"></v-color-picker>
                      </v-dialog>
                    </v-col>
                  </v-row>
                </v-container>
                <small>*為必填欄位</small>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  color="blue darken-1"
                  text
                  @click.prevent="resetNewEvent"
                >取消</v-btn>
                <v-btn
                  color="blue darken-1"
                  text
                  @click.prevent="addEvent"
                >新增</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-menu
            bottom
            right
          >

            <template v-slot:activator="{ on }">
              <v-btn
                outlined
                color="grey darken-2"
                v-on="on"
              >
                <span>{{ typeToLabel[type] }}</span>
                <v-icon right>mdi-menu-down</v-icon>
              </v-btn>
            </template>
            <v-list>
              <v-list-item @click="type = 'day'">
                <v-list-item-title>日</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = 'week'">
                <v-list-item-title>週</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = 'month'">
                <v-list-item-title>月</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = '4day'">
                <v-list-item-title>4 日</v-list-item-title>
              </v-list-item>
            </v-list>
          </v-menu>
        </v-toolbar>
      </v-sheet>
      <v-sheet height="600">
        <v-calendar
          ref="calendar"
          v-model="focus"
          color="primary"
          :events="events"
          :event-color="getEventColor"
          :now="today"
          :type="type"
          locale="zh-tw"
          @click:event="showEvent"
          @click:more="viewDay"
          @click:date="viewDay"
          @change="updateRange"
        ></v-calendar>
        <v-menu
          v-model="selectedOpen"
          :close-on-content-click="false"
          :activator="selectedElement"
          offset-x
        >
          <v-card
            color="grey lighten-4"
            min-width="350px"
            flat
          >
            <v-toolbar
              :color="selectedEvent.color"
              dark
            >
              <v-btn
                icon
                @click.prevent="deleteEvent(selectedEvent)"
              >
                <v-icon>mdi-delete</v-icon>
              </v-btn>
              <v-toolbar-title v-html="selectedEvent.name"></v-toolbar-title>
              <v-spacer></v-spacer>
            </v-toolbar>
            <v-card-text v-if="currentlyEditing !== selectedEvent.id">
              <span v-html="selectedEvent.details"></span>
            </v-card-text>
            <v-textarea
              v-else
              filled
              label="描述"
              v-model="selectedEvent.details"
            ></v-textarea>
            <v-card-actions>
              <v-btn
                text
                color="secondary"
                @click="selectedOpen = false"
              >
                取消
              </v-btn>
              <v-btn
                text
                color="primary"
                v-if="currentlyEditing !== selectedEvent.id"
                @click.prevent="currentlyEditing = selectedEvent.id"
              >
                修改
              </v-btn>
              <v-btn
                v-else
                text
                color="primary"
                @click.prevent="updateEvent(selectedEvent)"
              >
                確認修改
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-menu>
      </v-sheet>
    </v-col>
  </v-row>
</template>

<script>
  import { db } from "@/db";

  export default {
    data: () => ({
      today: new Date().toISOString().substr(0, 10),
      focus: new Date().toISOString().substr(0, 10),
      type: "month",
      typeToLabel: {
        month: "月",
        week: "週",
        day: "日",
        "4day": "4 日"
      },
      newEvent: {
        name: null,
        details: null,
        start: new Date().toISOString().substr(0, 10),
        end: new Date().toISOString().substr(0, 10),
        color: "#ffffff"
      },
      start: null,
      end: null,
      currentlyEditing: null,
      selectedEvent: {},
      selectedElement: null,
      selectedOpen: false,
      events: [],
      color: "#ffffff",
      names: [
        "blue",
        "indigo",
        "deep-purple",
        "cyan",
        "green",
        "orange",
        "grey darken-1"
      ],
      dialog: false,
      colorDialog: false,
      datePickerStart: false,
      datePickerEnd: false,
      loading: false
    }),
    computed: {
      title() {
        const { start, end } = this;
        if (!start || !end) {
          return "";
        }

        const startMonth = this.monthFormatter(start);
        const endMonth = this.monthFormatter(end);
        // const suffixMonth = startMonth === endMonth ? "" : endMonth;

        const startYear = start.year + "年";
        const endYear = end.year;
        // const suffixYear = startYear === endYear ? "" : endYear;

        const startDay = start.day + "日";
        const endDay = end.day + "日";

        switch (this.type) {
          case "month":
            return `${startYear} ${startMonth}`;
          case "week":
          case "4day":
            return `${startYear} ${startMonth} ${startDay}  - ${endYear} ${endMonth} ${endDay} `;
          case "day":
            return `${startYear} ${startMonth} ${startDay}`;
        }
        return "";
      },
      monthFormatter() {
        return this.$refs.calendar.getFormatter({
          timeZone: "UTC",
          month: "long"
        });
      }
    },
    mounted() {
      this.getEvents();
    },
    methods: {
      async getEvents() {
        let snapshot = await db.collection("calEvent").get();
        let events = [];
        snapshot.forEach(doc => {
          let appData = doc.data();
          appData.id = doc.id;
          events.push(appData);
        });
        this.events = events;
      },
      async updateEvent(event) {
        await db
          .collection("calEvent")
          .doc(this.currentlyEditing)
          .update({
            details: event.details
          });
        this.selectedOpen = false;
        this.currentlyEditing = null;
      },
      async deleteEvent(event) {
        if (confirm(`確定要刪除 ${event.name} ?`)) {
          await db
            .collection("calEvent")
            .doc(event.id)
            .delete();
          this.selectedOpen = false;
          this.getEvents();
        }
      },
      async addEvent() {
        const { name, details, start, end, color } = this.newEvent;
        if (name && start && end) {
          this.loading = true;
          await db.collection("calEvent").add({
            name: name,
            details: details,
            start: start,
            end: end,
            color: color
          });
          this.resetNewEvent();
          this.getEvents();
          this.loading = false;
        } else {
          alert("請填寫必填欄位");
        }
      },
      resetNewEvent() {
        this.newEvent = {
          name: null,
          details: null,
          start: new Date().toISOString().substr(0, 10),
          end: new Date().toISOString().substr(0, 10),
          color: "#ffffff"
        };

        this.dialog = false;
      },
      viewDay({ date }) {
        this.focus = date;
        this.type = "day";
      },
      getEventColor(event) {
        return event.color;
      },
      setToday() {
        this.focus = this.today;
      },
      prev() {
        this.$refs.calendar.prev();
      },
      next() {
        this.$refs.calendar.next();
      },
      showEvent({ nativeEvent, event }) {
        const open = () => {
          this.selectedEvent = event;
          this.selectedElement = nativeEvent.target;
          setTimeout(() => (this.selectedOpen = true), 10);
        };

        if (this.selectedOpen) {
          this.selectedOpen = false;
          setTimeout(open, 10);
        } else {
          open();
        }

        nativeEvent.stopPropagation();
      },
      updateRange({ start, end }) {
        this.start = start;
        this.end = end;
      }
    }
  };
</script>

<style>
</style>