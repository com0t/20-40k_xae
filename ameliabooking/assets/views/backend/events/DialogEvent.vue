<template>
  <div class="am-dialog-event-inner">

    <!-- Dialog Loader -->
    <div class="am-dialog-loader" v-show="dialogLoading">
      <div class="am-dialog-loader-content">
        <img :src="$root.getUrl + 'public/img/spinner.svg'" class=""/>
        <p>{{$root.labels.loader_message}}</p>
      </div>
    </div>

    <!-- Dialog Content -->
    <div class="am-dialog-scrollable" :class="{'am-edit': event.id !== 0}" v-if="event && !dialogLoading">

      <!-- Dialog Header -->
      <div v-if="showHeader" class="am-dialog-header">
        <el-row>
          <el-col :span="18">
            <h2 v-if="event && event.id !== 0">{{$root.labels.edit_event}}</h2>
            <h2 v-else>{{$root.labels.new_event}}</h2>
          </el-col>
          <el-col :span="6" class="align-right">
            <el-button @click="closeDialog" class="am-dialog-close" size="small" icon="el-icon-close">
            </el-button>
          </el-col>
        </el-row>
      </div>


      <el-form v-if="mounted" :model="event" ref="event" label-position="top">
        <el-tabs v-model="defaultEventTab">

          <!-- Event Details -->
          <el-tab-pane :label="$root.labels.event_details" name="details">

            <!-- Event Name -->
            <el-form-item prop="name" :rules="rules.name" @input="clearValidation()">
              {{ $root.labels.event_name }}
              <div class="am-event-translate" @click="showDialogTranslate('name')" v-if="!isCabinet">
                <img class="am-dialog-translate-svg" width="16px" :src="$root.getUrl+'public/img/translate.svg'">
                {{ $root.labels.translate }}
              </div>
              <el-input v-model="event.name" :placeholder="$root.labels.enter_event_name" style="margin-top: 5px !important" @change="trimProperty(event, 'name')">
              </el-input>
            </el-form-item>

            <!-- Event Dates -->
            <div class="am-event-dates am-section-grey">
              <!-- Event Start -->
              <div
                v-for="(period, index) in event.periods"
                :key="index"
                class="am-event-date"
              >
                <!-- Date -->
                <el-row :gutter="10">
                  <el-col :sm="6">
                    <p>{{$root.labels.event_period_dates}}</p>
                  </el-col>
                  <el-col :sm="16" class="v-calendar-column">
                    <el-form-item :prop="'periods.' + index + '.range'" :rules="rules.range">
                      <v-date-picker
                        v-model="period.range"
                        mode='range'
                        style="margin-bottom: 16px;"
                        input-class="el-input__inner"
                        popover-visibility="focus"
                        popover-direction="bottom"
                        :popover-align="screenWidth < 768 ? 'center' : 'right'"
                        :show-day-popover=false
                        :attributes="[{
                          dates: {
                            start: getNowDate()
                          },
                          eventDateIndex: index
                        }]"
                        :is-expanded=false
                        :is-required=false
                        :is-read-only=true
                        :is-double-paned="false"
                        :tint-color='isCabinet ? $root.settings.customization.primaryColor : "#1A84EE"'
                        :input-props='{class: "el-input__inner", readOnly: "readonly"}'
                        :formats="vCalendarFormats"
                        :available-dates="{start: getNowDate()}"
                      >
                      </v-date-picker>
                    </el-form-item>

                  </el-col>
                </el-row>

                <!-- Time -->
                <el-row :gutter="10">
                  <el-col :sm="6">
                    <p>{{$root.labels.event_period_time}}</p>
                  </el-col>
                  <el-col :sm="8" class="am-event-period-start">
                    <el-form-item :prop="'periods.' + index + '.startTime'" :rules="rules.startTime">
                      <el-time-select
                        v-model="period.startTime"
                        :picker-options="getTimeSelectOptionsWithLimits(null, isCabinet ? null : period.endTime)"
                        :popper-class="'am-dropdown-cabinet'"
                        size="large"
                      >
                      </el-time-select>
                    </el-form-item>
                  </el-col>
                  <el-col :sm="8">
                    <el-form-item :prop="'periods.' + index + '.endTime'" :rules="rules.endTime">
                      <el-time-select
                        v-model="period.endTime"
                        :picker-options="getTimeSelectOptionsWithLimits(isCabinet ? null : period.startTime, null)"
                        :popper-class="'am-dropdown-cabinet'"
                        size="large"
                      >
                      </el-time-select>
                    </el-form-item>
                  </el-col>
                </el-row>

                <!-- Zoom -->
                <el-row v-if="$root.settings.zoom.enabled && period.zoomMeeting" :gutter="10">
                  <el-col v-if="$root.settings.role !== 'customer'" :sm="12">
                    <p>{{ $root.labels.zoom_start_link }}</p>
                  </el-col>
                  <el-col v-if="$root.settings.role !== 'customer'" :sm="12">
                    <p><a class="am-link" :href="period.zoomMeeting.startUrl">{{ $root.labels.zoom_click_to_start }}</a>
                    </p>
                  </el-col>
                  <el-col :sm="12">
                    <p>{{ $root.labels.zoom_join_link }}</p>
                  </el-col>
                  <el-col :sm="12">
                    <p><a class="am-link" :href="period.zoomMeeting.joinUrl">{{ $root.labels.zoom_click_to_join }}</a></p>
                  </el-col>
                </el-row>

                <!-- Lesson Space -->
                <el-row v-if="$root.settings.lessonSpace.enabled && period.lessonSpace" :gutter="10">
                  <el-col :sm="12">
                    <p>{{ $root.labels.lesson_space_link }}</p>
                  </el-col>
                  <el-col :sm="12">
                    <p><a class="am-link" :href="period.lessonSpace">{{ $root.labels.lesson_space_join }}</a>
                    </p>
                  </el-col>
                </el-row>

                <!-- Delete Event Date -->
                <div
                  v-show="event.periods.length > 1 && period.bookings.length === 0"
                  class="am-delete-element disabled"
                  @click="deleteEventDate(index)"
                >
                  <i class="el-icon-minus"></i>
                </div>

              </div>

              <div class="am-add-event-date">
                <el-button size="small" type="primary" @click="addEventDate()">{{$root.labels.add_date}}</el-button>
              </div>
            </div>

            <!-- Recurring -->
            <el-popover :disabled="!$root.isLite" ref="recurringPop" v-bind="$root.popLiteProps"><PopLite/></el-popover>
            <div class="am-section-grey" :class="{'am-lite-disabled': ($root.isLite)}" v-popover:recurringPop>
              <el-checkbox v-model="event.isRecurring" :disabled="$root.isLite">{{$root.labels.event_recurring_enabled}}</el-checkbox>
              <div class="am-recurring-event" v-if="event.isRecurring && !$root.isLite">
                <el-row :gutter="10">
                  <el-col :lg="10" :md="10" :sm="24">
                    <p>{{$root.labels.event_recurring_period}}</p>
                  </el-col>
                  <el-col :lg="14" :md="14" :sm="24">
                    <el-form-item prop="recurringCycle" :rules="rules.recurringCycle">
                      <el-select
                        v-model="event.recurring.cycle"
                        clearable
                        filterable
                        :style="{ backgroundColor: activeColor }"
                        :placeholder="$root.labels.select_repeat_period"
                        :popper-class="'am-dropdown-cabinet'"
                        :disabled="!(event.id === 0 || (event.id !== 0 && originRecurring.cycle === null))"
                        @change="setRecurringValues"
                      >
                        <el-option
                          v-for="period in recurringPeriods"
                          :key="period.value"
                          :label="period.label"
                          :value="period.value"
                        >
                        </el-option>
                      </el-select>
                    </el-form-item>
                  </el-col>
                </el-row>

                <el-row :gutter="10">
                  <el-col :lg="10" :md="10" :sm="24">
                    <p>{{$root.labels.recurring_every}}</p>
                  </el-col>
                  <el-col :lg="14" :md="14" :sm="24">
                    <el-form-item prop="recurringInterval" :rules="rules.recurringInterval">
                      <el-select
                        v-model="event.recurring.cycleInterval"
                        clearable
                        filterable
                        :style="{ backgroundColor: activeColor }"
                        :disabled="!(event.id === 0 || (event.id !== 0 && originRecurring.cycle === null))"
                        :popper-class="'am-dropdown-cabinet'"
                        :placeholder="repeatIntervals.length>0 ? $root.labels.select_repeat_interval : $root.labels.select_repeat_period"
                      >
                        <el-option
                          v-for="(item, index) in repeatIntervals"
                          :key="index"
                          :label="item.label"
                          :value="item.value"
                        >
                        </el-option>
                      </el-select>
                    </el-form-item>
                  </el-col>
                </el-row>


                <el-row :gutter="10" v-if="(event.id === 0 && event.recurring.cycle === 'monthly') || (event.id !== 0 && originRecurring.monthlyRepeat === 'each' && originRecurring.cycle === 'monthly')">
                  <el-col :lg="10" :md="10" :sm="24" class="text-center">
                    <el-radio v-model="monthlyRepeat" label="each" v-if="event.id === 0 || (event.id !== 0 && originRecurring.cycle === null)">
                      {{ $root.labels.recurring_each }}
                    </el-radio>
                    <p v-else>
                      {{ $root.labels.recurring_each }}
                    </p>
                  </el-col>
                  <el-col :lg="14" :md="14" :sm="24" class="v-calendar-column">
                    <el-form-item>
                      <v-date-picker
                        v-if="monthlyRepeat === 'each'"
                        v-model="monthDateComp"
                        mode='single'
                        input-class="el-input__inner"
                        popover-visibility="focus"
                        :popover-direction="screenWidth < 768 ? 'bottom' : 'top'"
                        :popover-align="screenWidth < 768 ? 'center' : 'center'"
                        :tint-color='isCabinet ? $root.settings.customization.primaryColor : "#1A84EE"'
                        :is-expanded=false
                        :is-required=false
                        :is-double-paned="false"
                        :show-day-popover=false
                        :input-props='{class: "el-input__inner", readOnly: "readonly"}'
                        :formats="vCalendarFormats"
                        :min-date="calculateMinDate()"
                        :max-date="calculateMaxDate()"
                      >
                      </v-date-picker>
                      <el-input
                        v-else
                        :disabled="true"
                        :value="event.recurring && event.recurring.monthDate ? getFrontedFormattedDate(getDatabaseFormattedDate(event.recurring.monthDate)) : ''"
                      />
                    </el-form-item>
                  </el-col>
                </el-row>

                <el-row :gutter="10" v-if="(event.id === 0 && event.recurring.cycle === 'monthly') || (event.id !== 0 && originRecurring.monthlyRepeat === 'on' && originRecurring.cycle === 'monthly')">
                  <el-col :lg="10" :md="10" :sm="24">
                    <el-radio v-model="monthlyRepeat" label="on" v-if="event.id === 0 || (event.id !== 0 && originRecurring.cycle === null)">
                      {{ $root.labels.recurring_on }}
                    </el-radio>
                    <p v-else>{{ $root.labels.recurring_on }}</p>
                  </el-col>
                  <el-col :lg="14" :md="14" :sm="24" class="text-center">
                    <el-row :gutter="10">
                      <el-col :lg="12" :md="12" :sm="24">
                        <el-form-item>
                          <el-select
                            v-model="monthlyOnRepeat"
                            clearable
                            filterable
                            :placeholder="monthlyWeekDayRepeat[0].label"
                            :popper-class="'am-dropdown-cabinet'"
                            :disabled="monthlyRepeat === 'each' || !(event.id === 0 || (event.id !== 0 && originRecurring.cycle === null))"
                            :style="{ backgroundColor: activeColor }"
                          >
                            <el-option
                              v-for="weekDayRepeat in monthlyWeekDayRepeat"
                              :key="weekDayRepeat.value"
                              :label="weekDayRepeat.label"
                              :value="weekDayRepeat.value"
                            >
                            </el-option>
                          </el-select>
                        </el-form-item>
                      </el-col>
                      <el-col :lg="12" :md="12" :sm="24">
                        <el-form-item>
                          <el-select
                            v-model="monthlyOnDay"
                            clearable
                            filterable
                            :placeholder="weekDays[0].label"
                            :popper-class="'am-dropdown-cabinet'"
                            :disabled="monthlyRepeat === 'each' || !(event.id === 0 || (event.id !== 0 && originRecurring.cycle === null))"
                            :style="{ backgroundColor: activeColor }"
                          >
                            <el-option
                              v-for="weekDay in weekDays"
                              :key="weekDay.value"
                              :label="weekDay.label"
                              :value="weekDay.value"
                            >
                            </el-option>
                          </el-select>
                        </el-form-item>
                      </el-col>
                    </el-row>
                  </el-col>
                </el-row>

                <el-row :gutter="10">
                  <el-col :lg="10" :md="10" :sm="24">
                    <p>{{$root.labels.event_recurring_until}}</p>
                  </el-col>
                  <el-col :lg="14" :md="14" :sm="24" class="v-calendar-column">
                    <el-form-item prop="recurringUntilDate" :rules="rules.recurringUntilDate">
                      <v-date-picker
                        v-model="event.recurring.until"
                        mode='single'
                        input-class="el-input__inner"
                        popover-visibility="focus"
                        :is-expanded=false
                        :is-required=false
                        :is-double-paned=false
                        :show-day-popover=false
                        :popover-direction="screenWidth < 768 ? 'bottom' : 'top'"
                        :popover-align="screenWidth < 768 ? 'center' : 'center'"
                        :tint-color='isCabinet ? $root.settings.customization.primaryColor : "#1A84EE"'
                        :input-props='{class: "el-input__inner", readOnly: "readonly"}'
                        :formats="vCalendarFormats"
                        :available-dates="{start: event.id === 0 ? getNowDate() : originRecurring.until}"
                        @dayclick="changeBookingEndsDate"
                      >
                      </v-date-picker>
                    </el-form-item>
                  </el-col>
                </el-row>
              </div>
            </div>

            <!-- Booking Dates -->
            <div class="am-section-grey">
              <el-checkbox v-model="event.bookingStartsNow">{{$root.labels.event_booking_opens_now}}</el-checkbox>

              <div class="am-booking-starts" v-show="!event.bookingStartsNow">
                <el-row :gutter="10">

                  <el-col :sm="24">
                    <label class="el-form-item__label">{{$root.labels.event_booking_opens_on}}</label>
                  </el-col>

                  <el-col :sm="15" class="v-calendar-column">
                    <el-form-item prop="bookingStartsDate" :rules="rules.bookingStartsDate">
                      <v-date-picker
                        v-model="event.bookingStartsDate"
                        :is-double-paned="false"
                        mode='single'
                        input-class="el-input__inner"
                        popover-visibility="focus"
                        popover-direction="bottom"
                        :popover-align="screenWidth < 768 ? 'center' : 'left'"
                        :is-expanded=false
                        :is-required=false
                        :show-day-popover=false
                        :tint-color='isCabinet ? $root.settings.customization.primaryColor : "#1A84EE"'
                        :input-props='{class: "el-input__inner", readOnly: "readonly"}'
                        :formats="vCalendarFormats"
                        :available-dates="{start: getNowDate()}"
                        @dayclick="changeBookingStartsDate"
                      >
                      </v-date-picker>
                    </el-form-item>

                  </el-col>
                  <el-col :sm="9">
                    <el-form-item prop="bookingStartsTime" :rules="rules.bookingStartsTime">
                      <el-time-select
                        v-model="event.bookingStartsTime"
                        size="large"
                        :popper-class="'am-dropdown-cabinet'"
                        :picker-options="getTimeSelectOptionsWithLimits(null, null)"
                      >
                      </el-time-select>
                    </el-form-item>
                  </el-col>
                </el-row>
              </div>
              <div v-if="event.isRecurring">
                <el-checkbox
                  v-model="event.bookingOpensRec"
                  true-label="same"
                  false-label="calculate"
                >
                  {{$root.labels.apply_to_all}}
                </el-checkbox>
                <el-tooltip placement="top">
                  <div slot="content" v-html="$root.labels.event_booking_opens_apply"></div>
                  <i class="el-icon-question am-tooltip-icon"></i>
                </el-tooltip>
              </div>
            </div>

            <div class="am-section-grey">
              <el-checkbox v-model="event.bookingEndsAfter">{{$root.labels.event_booking_closes_after}}</el-checkbox>

              <div class="am-booking-ends" v-show="!event.bookingEndsAfter">
                <el-row :gutter="10">

                  <el-col :sm="24">
                    <label class="el-form-item__label">{{$root.labels.event_booking_closes_on}}</label>
                  </el-col>

                  <el-col :sm="15" class="v-calendar-column">
                    <el-form-item prop="bookingEndsDate" :rules="rules.bookingEndsDate">
                      <v-date-picker
                        v-model="event.bookingEndsDate"
                        mode='single'
                        input-class="el-input__inner"
                        popover-visibility="focus"
                        popover-direction="bottom"
                        :popover-align="screenWidth < 768 ? 'center' : 'left'"
                        :is-double-paned="false"
                        :is-required=false
                        :is-expanded=false
                        :show-day-popover=false
                        :tint-color='isCabinet ? $root.settings.customization.primaryColor : "#1A84EE"'
                        :input-props='{class: "el-input__inner", readOnly: "readonly"}'
                        :formats="vCalendarFormats"
                        :available-dates="{start: getNowDate()}"
                        @dayclick="changeBookingEndsDate"
                      >
                      </v-date-picker>
                    </el-form-item>
                  </el-col>

                  <el-col :sm="9">
                    <el-form-item prop="bookingEndsTime" :rules="rules.bookingEndsTime">
                      <el-time-select
                        v-model="event.bookingEndsTime"
                        :popper-class="'am-dropdown-cabinet'"
                        :picker-options="getTimeSelectOptionsWithLimits(null, null)"
                        size="large"
                      >
                      </el-time-select>
                    </el-form-item>
                  </el-col>
                </el-row>
              </div>
              <div v-if="event.isRecurring">
                <el-checkbox
                  v-model="event.bookingClosesRec"
                  true-label="same"
                  false-label="calculate"
                >
                  {{$root.labels.apply_to_all}}
                </el-checkbox>
                <el-tooltip placement="top">
                  <div slot="content" v-html="$root.labels.event_booking_closes_apply"></div>
                  <i class="el-icon-question am-tooltip-icon"></i>
                </el-tooltip>
              </div>
            </div>

            <!-- Slots & Price -->
            <div class="am-border-bottom">
              <el-row :gutter="10">
                <el-col :lg="12" :md="12" :sm="24">
                  <p>{{$root.labels.price}}</p>
                </el-col>
                <el-col :lg="12" :md="12" :sm="24">
                  <money v-model="event.price" v-bind="moneyComponentData" @input="priceChanged" class="el-input el-input__inner">
                  </money>
                </el-col>
              </el-row>
              <el-popover :disabled="!$root.isLite" ref="depositPop" v-bind="$root.popLiteProps"><PopLite/></el-popover>
              <div class="am-setting-box am-switch-box" v-if="parseFloat(event.price) > 0" v-popover:depositPop :class="{'am-lite-disabled': ($root.isLite)}">
                <!-- Deposit Enabled -->
                <el-row type="flex" align="middle" :gutter="24">
                  <el-col :span="19">
                    {{ $root.labels.deposit_enabled }}
                  </el-col>
                  <el-col :span="5" class="align-right">
                    <el-switch
                      v-model="depositEnabled"
                      active-text=""
                      inactive-text=""
                      @change="depositEnabledChanged"
                      :disabled="$root.isLite"
                    >
                    </el-switch>
                  </el-col>
                </el-row>

                <el-row :gutter="24" v-if="depositEnabled" class="am-service-deposit">
                  <el-col :span="12">
                    <el-form-item>
                      <label :slot="'label'">
                        {{ $root.labels.deposit_payment }}:
                        <el-tooltip placement="top">
                          <div slot="content" v-html="$root.labels.deposit_payment_tooltip"></div>
                          <i class="el-icon-question am-tooltip-icon"></i>
                        </el-tooltip>
                      </label>
                      <el-select
                        v-model="depositPayment"
                        placeholder=""
                        :popper-class="'am-dropdown-cabinet'"
                        @change="depositChanged()"
                      >
                        <el-option
                          v-for="(item, index) in depositOptions"
                          :key="index"
                          :label="item.label"
                          :value="item.value"
                        >
                        </el-option>
                      </el-select>
                    </el-form-item>
                  </el-col>

                  <el-col :span="12">
                    <el-form-item :label="$root.labels.deposit_amount + (depositPayment === 'fixed' ? ' (' + getCurrencySymbol() + ')' : '') + (depositPayment === 'percentage' ? ' (%)' : '') +  ':'" prop="deposit" :rules="rules.deposit">
                      <div v-if="depositPayment === 'fixed'" class="el-input">
                        <money v-model="event.deposit" v-bind="moneyComponentData" @input="depositChanged" class="el-input__inner"></money>
                      </div>

                      <el-input-number
                        v-if="depositPayment === 'percentage'"
                        v-model="event.deposit"
                        :min="0"
                        :max="100"
                        @input="depositChanged()"
                      >
                      </el-input-number>
                    </el-form-item>
                  </el-col>
                </el-row>

                <el-row v-if="event.maxCapacity > 1 && depositPayment === 'fixed' && depositEnabled">
                  <el-col>
                    <el-checkbox v-model="event.depositPerPerson">
                      {{$root.labels.deposit_per_person}}
                      <el-tooltip placement="top">
                        <div slot="content" v-html="$root.labels.deposit_by_the_number_of_people"></div>
                        <i class="el-icon-question am-tooltip-icon"></i>
                      </el-tooltip>
                    </el-checkbox>
                  </el-col>
                </el-row>

                <el-row class="am-event-deposit" :gutter="24" v-if="depositEnabled">
                  <el-col>
                    <el-checkbox v-model="event.fullPayment">
                      {{$root.labels.allow_total_amount}}
                      <el-tooltip placement="top">
                        <div slot="content" v-html="$root.labels.allow_customers_to_pay_total"></div>
                        <i class="el-icon-question am-tooltip-icon"></i>
                      </el-tooltip>
                    </el-checkbox>
                    <hr>
                  </el-col>
                </el-row>

                <el-row :gutter="24" v-if="depositEnabled" class="am-service-deposit">
                  <el-col :span="24">
                    <i class="el-icon-warning-outline"></i>
                    <label>{{$root.labels.deposit_info}}</label>
                  </el-col>
                </el-row>
              </div>
              <el-row :gutter="10">
                <el-col :lg="12" :md="12" :sm="24">
                  <p>{{$root.labels.event_max_capacity}}</p>
                </el-col>
                <el-col :lg="12" :md="12" :sm="24">
                  <el-input-number
                    v-model="event.maxCapacity"
                    :min="1"
                  >
                  </el-input-number>
                </el-col>
              </el-row>
              <el-row :gutter="10" v-if="event.maxCapacity > 1">
                <el-col :lg="24" :md="24" :sm="24">
                  <el-checkbox v-model="event.bringingAnyone">{{$root.labels.event_bringing_anyone}}</el-checkbox>
                </el-col>
              </el-row>
              <el-row :gutter="10" v-if="event.maxCapacity > 1">
                <el-col :lg="24" :md="24" :sm="24">
                  <el-checkbox v-model="event.bookMultipleTimes">{{$root.labels.event_book_more_than_once}}</el-checkbox>
                </el-col>
              </el-row>
            </div>

            <!-- Address -->
            <div class="am-border-bottom">

              <el-row :gutter="10">
                <el-col :lg="12" :md="12" :sm="24">
                  <p>{{$root.labels.event_select_address}}</p>
                </el-col>
                <el-col :lg="12" :md="12" :sm="24">
                  <el-select
                    v-model="event.locationId"
                    :placeholder="$root.labels.select"
                    :popper-class="'am-dropdown-cabinet'"
                    :clearable="true"
                    :value="null"
                  >
                    <el-option
                      :key="null"
                      :label="this.$root.labels.event_custom_address"
                      :value="null"
                    >
                    </el-option>
                    <el-option
                      v-for="location in locations"
                      :key="location.id"
                      :label="location.name"
                      :value="location.id"
                    >
                    </el-option>
                  </el-select>
                </el-col>
              </el-row>
              <el-row :gutter="10" v-show="!event.locationId">
                <el-col :lg="12" :md="12" :sm="24">
                  <p>{{$root.labels.event_custom_address}}</p>
                </el-col>
                <el-col :lg="12" :md="12" :sm="24">
                  <el-input v-model="event.customLocation" :placeholder="$root.labels.enter_address">
                  </el-input>
                </el-col>
              </el-row>

            </div>

            <!-- Employee & Tags -->
            <div class="am-border-bottom">

              <el-row :gutter="10" v-if="$root.settings.zoom.enabled">
                <el-col :lg="12" :md="12" :sm="24">
                  <p>{{ $root.labels.zoom_user }}</p>
                </el-col>
                <el-col :lg="12" :md="12" :sm="24">
                  <!-- Zoom Users List -->
                  <el-select
                    v-model="event.zoomUserId"
                    clearable
                    filterable
                    :placeholder="$root.labels.zoom_user_placeholder"
                    :popper-class="'am-dropdown-cabinet'"
                    @change="clearValidation()"
                  >
                    <el-option
                      v-for="(zoomUser, index) in zoomUsers"
                      :key="index"
                      :label="zoomUser.first_name + ' ' + zoomUser.last_name + ' (' + zoomUser.email + ')'"
                      :value="zoomUser.id"
                    >
                    </el-option>
                  </el-select>
                </el-col>
              </el-row>

              <el-row :gutter="10" v-if="canManage()">
                <el-col :lg="12" :md="12" :sm="24">
                  <p>{{$root.labels.event_staff}}</p>
                </el-col>
                <el-col :lg="12" :md="12" :sm="24">
                  <!-- Recurring -->
                  <el-select
                    v-model="event.providers"
                    multiple
                    value-key="id"
                    :placeholder="$root.labels.select"
                    :popper-class="'am-dropdown-cabinet'"
                  >
                    <el-option
                      v-for="emp in employees"
                      :key="emp.id"
                      :label="emp.firstName + ' ' + emp.lastName"
                      :value="emp"
                    >
                    </el-option>
                  </el-select>
                </el-col>
              </el-row>

              <el-row :gutter="10">
                <el-col :lg="12" :md="12" :sm="24">
                  <p>{{$root.labels.event_tags}}</p>
                </el-col>
                <el-col :lg="12" :md="12" :sm="24">
                  <el-popover :disabled="!$root.isLite" ref="tagsPop" v-bind="$root.popLiteProps"><PopLite/></el-popover>
                  <el-select
                    v-model="event.tags"
                    multiple
                    filterable
                    allow-create
                    default-first-option
                    :placeholder="$root.labels.event_tags_select_or_create"
                    :popper-class="'am-dropdown-cabinet'"
                    :no-data-text="$root.labels.event_tags_create"
                    @change="tagsChanged"
                    :disabled="$root.isLite"
                    v-popover:tagsPop
                  >
                    <el-option
                      v-for="(tag, index) in tags"
                      v-if="tag"
                      :key="index"
                      :label="tag"
                      :value="tag"
                    >
                    </el-option>
                  </el-select>
                </el-col>
              </el-row>

            </div>

            <!-- Description -->
            <content-block
              :label="$root.labels.description"
              :entity="event"
              textProperty="description"
              htmlProperty="descriptionHtml"
              :hasTranslation="!isCabinet"
              :hasQuill="!isCabinet"
              :textModeProp="true"
              :allowImage="false"
              @showDialogTranslate="showDialogTranslate('description')"
            >
            </content-block>

          </el-tab-pane>

          <!-- Customize -->
          <el-tab-pane :label="$root.labels.customize" name="customize">
            <BlockLite/>
            <!-- Gallery -->
            <gallery
              v-if="showGallery"
              :gallery="event.gallery"
              :label="$root.labels.event_gallery"
              @galleryUpdated="galleryUpdated"
              :class="{'am-lite-disabled': $root.isLite, 'am-lite-container-disabled': $root.isLite}"
            />

            <!-- Colors -->
            <div class="am-event-colors" :class="{'am-lite-disabled': $root.isLite, 'am-lite-container-disabled': $root.isLite}">
              <div class="am-event-section-title">
                {{$root.labels.event_colors}}
              </div>
              <div class="am-event-color-selection">
                <div>
                  <el-radio v-model="event.colorType" :label="1" value=1>{{$root.labels.event_colors_preset}}</el-radio>
                  <div class="am-event-swatches am-event-swatches-first">
                    <span
                      v-for="color in colors"
                      :key="color"
                      :class="{'color-active' : color === event.selectedColor}"
                      @click="changeEventColor"
                      :data-color="color"
                      :style="'background-color: ' + color"
                    >
                    </span>
                  </div>
                </div>
                <div>
                  <el-radio v-model="event.colorType" :label="2" value=2>{{$root.labels.event_colors_custom}}</el-radio>
                  <el-input
                    :disabled="event.colorType === 1"
                    v-model="event.customColor"
                    class="am-event-custom-color"
                    placeholder="000000"
                  >
                  </el-input>
                  <div class="am-event-swatches">
                    <span
                      :data-color="event.customColor"
                      :style="'background-color: ' + event.customColor"
                    >
                    </span>
                  </div>
                </div>
              </div>
            </div>

            <!-- Show/Hide Event -->
            <div :class="{'am-lite-disabled': $root.isLite, 'am-lite-container-disabled': $root.isLite}">
              <el-checkbox v-model="event.show">{{$root.labels.event_show_on_site}}</el-checkbox>
            </div>

          </el-tab-pane>

          <el-tab-pane :label="$root.labels.settings" name="settings">
            <BlockLite/>
            <entity-settings
              :settings="settings"
              :paymentsSettings="event.settings.payments"
              :generalSettings="event.settings.general"
              :lessonSpaceSettings="event.settings.lessonSpace"
            >
            </entity-settings>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </div>

    <!-- Dialog Actions -->
    <dialog-actions
        v-if="event && !dialogLoading && this.$root.settings.role !== 'customer'"
        formName="event"
        urlName="events"
        :isNew="event.id === 0"
        :entity="event"
        :getParsedEntity="getParsedEntity"
        :haveSaveConfirmation="haveSaveConfirmation"
        @validationFailCallback="validationFailCallback"
        @errorCallback="errorCallback"
        :hasIcons="false"
        :updateStash="true"
        :hasApplyGloballyVisibility="originRecurring.until"
        :hasApplyGloballyDeletion="originRecurring.until && event.status === 'rejected'"

        :status="{
          on: 'approved',
          off: 'rejected'
        }"

        :buttonType="{
          status: event.status === 'approved' ? 'danger' : 'primary',
          remove: 'danger',
          duplicate: 'primary'
        }"

        :buttonText="{
          action: {
            remove: $root.labels.event_delete,
            status: event.status === 'approved' ? $root.labels.event_cancel : $root.labels.event_open,
            duplicate: $root.labels.event_duplicate
          },
          confirm: {
            save: originRecurring.until ? {
              yes: $root.labels.update_following,
              no: $root.labels.save_single
            } : null,
            status: originRecurring.until ? {
              yes: event.status === 'rejected' ? $root.labels.open_following : $root.labels.cancel_following,
              no: $root.labels.save_single
            } : null,
            remove: originRecurring.until ? {
              yes: $root.labels.delete_following,
              no: $root.labels.save_single
            } : null
          }
        }"

        :action="{
          haveAdd: true,
          haveEdit: true,
          haveStatus: canManage(),
          haveRemove: $root.settings.capabilities.canDelete === true && event.status === 'rejected',
          haveRemoveEffect: event.status !== 'rejected',
          haveDuplicate: event.status === 'approved'
        }"

        :message="{
          success: {
            save: $root.labels.event_saved,
            remove: $root.labels.event_deleted,
            show: $root.labels.event_opened,
            hide: $root.labels.event_canceled
          },
          confirm: {
            save: originRecurring.until ? $root.labels.confirm_save_following : null,
            remove: originRecurring.until ? $root.labels.confirm_delete_following : $root.labels.confirm_delete,
            show: originRecurring.until ? $root.labels.confirm_open_following : $root.labels.confirm_open,
            hide: originRecurring.until ? $root.labels.confirm_cancel_following : $root.labels.confirm_cancel,
            duplicate: $root.labels.confirm_duplicate_event
          }
        }"
    >
    </dialog-actions>

  </div>

</template>

<script>
  import cabinetMixin from '../../../js/frontend/mixins/cabinetMixin'
  import dateMixin from '../../../js/common/mixins/dateMixin'
  import DialogActions from '../parts/DialogActions.vue'
  import durationMixin from '../../../js/common/mixins/durationMixin'
  import EntitySettings from '../parts/EntitySettings.vue'
  import Gallery from '../parts/Gallery.vue'
  import imageMixin from '../../../js/common/mixins/imageMixin'
  import notifyMixin from '../../../js/backend/mixins/notifyMixin'
  import priceMixin from '../../../js/common/mixins/priceMixin'
  import windowMixin from '../../../js/backend/mixins/windowMixin'
  import helperMixin from '../../../js/backend/mixins/helperMixin'
  import recurringMixin from '../../../js/common/mixins/recurringMixin'
  import { Money } from 'v-money'
  import moment from 'moment'
  import ContentBlock from '../parts/ContentBlock'

  export default {

    mixins: [
      cabinetMixin,
      dateMixin,
      durationMixin,
      imageMixin,
      notifyMixin,
      priceMixin,
      windowMixin,
      helperMixin,
      recurringMixin
    ],

    props: {
      selectedTimeZone: '',
      event: null,
      employees: null,
      locations: null,
      tags: null,
      settings: null,
      isCabinet: {
        required: false,
        default: false,
        type: Boolean
      },
      showHeader: {
        required: false,
        default: true,
        type: Boolean
      },
      showGallery: {
        required: false,
        default: true,
        type: Boolean
      }
    },

    data () {
      let validateNonNegativePrice = (rule, price, callback) => {
        if (price < 0) {
          callback(new Error(this.$root.labels.enter_non_negative_price_warning))
        } else {
          callback()
        }
      }

      let validatePositiveValue = (rule, price, callback) => {
        if (price <= 0) {
          callback(new Error(this.$root.labels.enter_positive_price_warning))
        } else {
          callback()
        }
      }

      let isRecurringUntilDateDateRequired = (rule, input, callback) => {
        if (!this.event.recurring.until && !input) {
          callback(new Error(this.$root.labels.select_date_warning))
        } else {
          callback()
        }
      }

      let isRecurringCycleDateRequired = (rule, input, callback) => {
        if (!this.event.recurring.cycle && !input) {
          callback(new Error(this.$root.labels.select_cycle_warning))
        } else {
          callback()
        }
      }

      let isRecurringIntervalRequired = (rule, input, callback) => {
        if (!this.event.recurring.cycleInterval && !input) {
          callback(new Error(this.$root.labels.select_interval_warning))
        } else {
          callback()
        }
      }

      let isBookingStartsDateRequired = (rule, input, callback) => {
        if (!this.event.bookingStartsNow && !input) {
          callback(new Error(this.$root.labels.select_date_warning))
        } else {
          callback()
        }
      }

      let isBookingStartsTimeRequired = (rule, input, callback) => {
        if (!this.event.bookingStartsNow && !input) {
          callback(new Error(this.$root.labels.select_time_warning))
        } else {
          callback()
        }
      }

      let isBookingEndsDateRequired = (rule, input, callback) => {
        if (!this.event.bookingEndsAfter && !input) {
          callback(new Error(this.$root.labels.select_date_warning))
        } else {
          callback()
        }
      }

      let isBookingEndsTimeRequired = (rule, input, callback) => {
        if (!this.event.bookingEndsAfter && !input) {
          callback(new Error(this.$root.labels.select_time_warning))
        } else {
          callback()
        }
      }

      return {
        monthDate: null,
        monthlyOnRepeat: 'first',
        monthlyOnDay: 'monday',
        monthlyRepeat: 'each',
        weekDays: [
          {
            label: this.$root.labels.weekday_monday,
            value: 'monday'
          },
          {
            label: this.$root.labels.weekday_tuesday,
            value: 'tuesday'
          },
          {
            label: this.$root.labels.weekday_wednesday,
            value: 'wednesday'
          },
          {
            label: this.$root.labels.weekday_thursday,
            value: 'thursday'
          },
          {
            label: this.$root.labels.weekday_friday,
            value: 'friday'
          },
          {
            label: this.$root.labels.weekday_saturday,
            value: 'saturday'
          },
          {
            label: this.$root.labels.weekday_sunday,
            value: 'sunday'
          }
        ],
        activeColor: 'white',
        repeatIntervals: [],
        depositEnabled: false,
        depositPayment: 'fixed',
        depositOptions: [
          {
            value: 'fixed',
            label: this.$root.labels.fixed_amount
          },
          {
            value: 'percentage',
            label: this.$root.labels.percentage
          }
        ],
        originRecurring: null,
        originPeriods: null,
        colors: [
          '#1788FB',
          '#4BBEC6',
          '#FBC22D',
          '#FA3C52',
          '#D696B8',
          '#689BCA',
          '#26CC2B',
          '#FD7E35',
          '#E38587',
          '#774DFB'
        ],
        monthlyWeekDayRepeat: [
          {
            label: this.$root.labels.recurring_date_first,
            value: 'first'
          },
          {
            label: this.$root.labels.recurring_date_second,
            value: 'second'
          },
          {
            label: this.$root.labels.recurring_date_third,
            value: 'third'
          },
          {
            label: this.$root.labels.recurring_date_fourth,
            value: 'fourth'
          },
          {
            label: this.$root.labels.recurring_date_last,
            value: 'last'
          }
        ],
        recurringPeriods: [
          {
            label: this.$root.labels.recurring_type_daily,
            value: 'daily'
          },
          {
            label: this.$root.labels.recurring_type_weekly,
            value: 'weekly'
          },
          {
            label: this.$root.labels.recurring_type_monthly,
            value: 'monthly'
          },
          {
            label: this.$root.labels.recurring_type_yearly,
            value: 'yearly'
          }
        ],
        zoomUsers: [],
        dialogLoading: true,
        executeUpdate: true,
        mounted: false,
        rules: {
          name: [
            {required: true, message: this.$root.labels.enter_name_warning, trigger: 'submit'}
          ],
          price: [
            {validator: validateNonNegativePrice, trigger: 'submit'},
            {required: true, message: this.$root.labels.enter_service_price_warning, trigger: 'submit', type: 'number'}
          ],
          deposit: [
            {validator: validatePositiveValue, trigger: 'submit'}
          ],
          range: [
            {
              required: true, message: this.$root.labels.select_date_warning, trigger: 'submit'
            }
          ],
          startTime: [
            {
              required: true, message: this.$root.labels.select_time_warning, trigger: 'submit'
            }
          ],
          endTime: [
            {
              required: true, message: this.$root.labels.select_time_warning, trigger: 'submit'
            }
          ],
          bookingStartsDate: [
            {
              validator: isBookingStartsDateRequired, trigger: 'submit'
            }
          ],
          bookingStartsTime: [
            {
              validator: isBookingStartsTimeRequired, trigger: 'submit'
            }
          ],
          bookingEndsDate: [
            {
              validator: isBookingEndsDateRequired, trigger: 'submit'
            }
          ],
          bookingEndsTime: [
            {
              validator: isBookingEndsTimeRequired, trigger: 'submit'
            }
          ],
          recurringUntilDate: [
            {
              validator: isRecurringUntilDateDateRequired, trigger: 'submit'
            }
          ],
          recurringCycle: [
            {
              validator: isRecurringCycleDateRequired, trigger: 'submit'
            }
          ],
          recurringInterval: [
            {
              validator: isRecurringIntervalRequired, trigger: 'submit'
            }
          ]
        },
        defaultEventTab: 'details'
      }
    },

    methods: {
      calculateMinDate () {
        if (this.event && this.event.periods && this.event.periods[0] && this.event.periods[0].range) {
          let date = this.event.periods[0].range.start
          return new Date(date.getFullYear(), date.getMonth(), 1)
        }
      },

      calculateMaxDate () {
        if (this.event && this.event.periods && this.event.periods[0] && this.event.periods[0].range) {
          let date = this.event.periods[0].range.start
          return new Date(date.getFullYear(), date.getMonth() + 1, 0)
        }
      },

      depositEnabledChanged () {
        if (this.depositEnabled) {
          this.event.depositPayment = this.depositPayment
        } else {
          this.event.depositPayment = 'disabled'
        }
      },

      depositChanged () {
        if (this.event.deposit > this.event.price && this.depositPayment === 'fixed') {
          this.event.deposit = this.event.price
        }
      },

      priceChanged () {
        if (this.event.deposit > this.event.price && this.depositPayment === 'fixed') {
          this.event.deposit = this.event.price
        }
      },

      canManage () {
        return this.$root.settings.role === 'admin' || this.$root.settings.role === 'manager'
      },

      galleryUpdated: function () {},

      validationFailCallback () {
        this.defaultEventTab = 'details'
      },

      tagsChanged: function () {},

      haveSaveConfirmation () {
        return this.event.id !== 0 && this.originRecurring.until
      },

      changeBookingStartsDate () {
        if (this.event.bookingStartsTime === null) {
          this.event.bookingStartsTime = '00:00'
        }
      },

      changeBookingEndsDate () {
        if (this.event.bookingEndsTime === null) {
          this.event.bookingEndsTime = '00:00'
        }
      },

      instantiateDialog () {
        if ((this.event !== null) && this.executeUpdate === true) {
          this.originPeriods = JSON.parse(JSON.stringify(this.event.periods))
          this.originRecurring = JSON.parse(JSON.stringify(this.event.recurring))
          this.setRecurringValues()
          if (this.event.recurring && this.event.recurring.cycle === 'monthly' && this.event.recurring.monthlyRepeat) this.monthlyRepeat = this.event.recurring.monthlyRepeat
          if (this.event.recurring && this.event.recurring.cycle === 'monthly' && this.event.recurring.monthlyOnRepeat) this.monthlyOnRepeat = this.event.recurring.monthlyOnRepeat
          if (this.event.recurring && this.event.recurring.cycle === 'monthly' && this.event.recurring.monthlyOnDay) this.monthlyOnDay = this.event.recurring.monthlyOnDay

          if (this.event.recurring && this.event.recurring.cycle === 'monthly' && this.event.recurring.monthlyRepeat === 'each') {
            if (!this.event.recurring.monthDate) {
              this.event.recurring.monthDate = new Date(moment(this.event.periods[0].range.start).format('YYYY-MM-DD'))
            } else {
              this.event.recurring.monthDate = new Date(moment(this.event.recurring.monthDate).format('YYYY-MM-DD'))
            }
          }

          if (this.event.periods[0].range && moment(this.event.periods[0].range.start).format('YYYY-MM-DD HH:mm:ss') === this.event.bookingCloses) this.event.bookingEndsAfter = true

          if (this.event.depositPayment === 'disabled') {
            this.depositEnabled = false
          } else {
            this.depositEnabled = true

            this.depositPayment = this.event.depositPayment
          }

          if (this.event.description !== null && this.event.description.startsWith('<!-- Content -->')) {
            this.event.descriptionHtml = this.event.description
          }

          this.mounted = true
          this.executeUpdate = false
          this.dialogLoading = false
        }

        if (this.event && this.isCabinet) {
          this.activeColor = 'transparent'
          if (this.event.depositPayment === 'disabled') {
            this.depositEnabled = false
          } else {
            this.depositPayment = this.event.depositPayment
          }

          if (this.isCabinet && this.event.description && this.event.description.startsWith('<!-- Content -->')) {
            this.event.description = this.event.description.replace('<!-- Content -->', '').replace(/(\r\n|\n|\r)/gm, '')
            this.event.descriptionHtml = this.event.description
          }
        }
      },

      getZoomUsers () {
        let config = null

        if (this.isCabinet) {
          config = Object.assign(this.getAuthorizationHeaderObject(), {params: {source: 'cabinet-' + this.$store.state.cabinet.cabinetType}})
        }

        this.$http.get(`${this.$root.getAjaxUrl}/zoom/users`, config)
          .then(response => {
            if ('data' in response.data && 'users' in response.data.data) {
              this.zoomUsers = response.data.data.users
            }
          })
          .catch(e => {
            this.notify(this.$root.labels.error, e.message, 'error')
          })
      },

      clearValidation () {
        if (typeof this.$refs.event !== 'undefined') {
          this.$refs.event.clearValidate()
        }
      },

      getParsedEntity (applyGlobally) {
        let $this = this

        let eventPeriods = []

        if (!this.originRecurring.until && this.event.isRecurring) applyGlobally = true

        this.event.periods.forEach(function (period, index) {
          let periodStart = $this.getDateString(period.range.start) + ' ' + period.startTime + ':00'
          let periodEnd = $this.getDateString(period.range.end) + ' ' + period.endTime + ':00'

          if ($this.selectedTimeZone === 'UTC') {
            periodStart = moment(periodStart, 'YYYY-MM-DD HH:mm').utc().format('YYYY-MM-DD HH:mm')
            periodEnd = moment(periodEnd, 'YYYY-MM-DD HH:mm').utc().format('YYYY-MM-DD HH:mm')
          }

          if (typeof $this.originPeriods[index] !== 'undefined') {
            period.id = $this.originPeriods[index].id
            period.eventId = $this.originPeriods[index].eventId
          }

          eventPeriods.push({
            id: period.id,
            eventId: $this.event.id ? $this.event.id : null,
            periodStart: periodStart,
            periodEnd: periodEnd
          })
        })

        let tags = []

        let eventSettings = JSON.parse(JSON.stringify(this.event.settings))

        if (eventSettings.payments.wc.productId === this.$root.settings.payments.wc.productId) {
          delete eventSettings.payments.wc
        }

        this.event.depositPayment = 'disabled'

        let monthDate = this.monthDate ? this.monthDate : (this.event.recurring.monthDate ? this.event.recurring.monthDate : null)

        return {
          id: this.event.id,
          parentId: this.event.parentId,
          name: this.event.name,
          periods: eventPeriods,
          utc: this.selectedTimeZone === 'UTC',
          timeZone: this.selectedTimeZone === 'UTC' ? null : this.selectedTimeZone,
          bookingOpens: !this.event.bookingStartsNow ? this.getDateString(this.event.bookingStartsDate) + ' ' + this.event.bookingStartsTime + ':00' : null,
          bookingCloses: !this.event.bookingEndsAfter ? this.getDateString(this.event.bookingEndsDate) + ' ' + this.event.bookingEndsTime + ':00' : null,
          bookingOpensRec: this.event.bookingOpensRec,
          bookingClosesRec: this.event.bookingClosesRec,
          recurring: null,
          bringingAnyone: this.event.bringingAnyone,
          bookMultipleTimes: this.event.bookMultipleTimes,
          maxCapacity: this.event.maxCapacity,
          price: this.event.price,
          tags: tags,
          providers: this.event.providers,
          description: this.event.description,
          gallery: this.event.gallery,
          color: this.event.colorType === 1 ? this.event.selectedColor : this.event.customColor,
          show: this.event.show,
          locationId: this.event.locationId !== null ? this.event.locationId : null,
          customLocation: this.event.locationId === null ? this.event.customLocation : null,
          applyGlobally: applyGlobally,
          settings: JSON.stringify(eventSettings),
          zoomUserId: this.event.zoomUserId,
          translations: this.event.translations,
          deposit: this.event.deposit,
          depositPayment: this.event.depositPayment,
          depositPerPerson: this.event.depositPerPerson,
          fullPayment: this.event.fullPayment
        }
      },

      errorCallback (responseData) {},

      closeDialog () {
        this.$emit('closeDialog')
      },

      addEventDate () {
        this.event.periods.push({
          id: null,
          eventId: null,
          range: {
            start: new Date(),
            end: new Date()
          },
          startTime: null,
          endTime: null,
          bookings: []
        })
      },

      deleteEventDate (dateKey) {
        this.event.periods.splice(dateKey, 1)
      },

      changeEventColor: function () {},

      showDialogTranslate: function (type) {
        this.$emit('showDialogTranslate', type)
      },

      setRecurringValues () {
        this.repeatIntervals = []
        let cycleLabel = ''
        let cycleLabelPlural = ''
        let repeatNumber = 6
        switch (this.event.recurring.cycle) {
          case ('daily'):
            cycleLabel = this.$root.labels.day
            cycleLabelPlural = this.$root.labels.days
            break

          case ('weekly'):
            cycleLabel = this.$root.labels.week
            cycleLabelPlural = this.$root.labels.weeks
            repeatNumber = 6
            break

          case ('monthly'):
            cycleLabel = this.$root.labels.month
            cycleLabelPlural = this.$root.labels.months
            repeatNumber = 12
            break

          case ('yearly'):
            cycleLabel = this.$root.labels.year
            cycleLabelPlural = this.$root.labels.years
            repeatNumber = 5
            break

          default:
            this.repeatIntervals = []
            return
        }
        this.repeatIntervals = this.getRepeatIntervalLabels(cycleLabel, cycleLabelPlural, repeatNumber)
      }
    },

    mounted () {
      this.instantiateDialog()

      if (this.$root.settings.zoom.enabled) {
        this.getZoomUsers()
      }
    },

    updated () {
      this.instantiateDialog()
    },

    components: {
      EntitySettings,
      DialogActions,
      ContentBlock,
      Money,
      Gallery
    },

    computed: {
      monthDateComp: {
        get: function () {
          if (this.event.recurring.monthDate) {
            return this.event.recurring.monthDate
          }
          if (!this.monthDate) {
            if (this.event && this.event.periods && this.event.periods[0] && this.event.periods[0].range) {
              this.monthDate = this.event.periods[0].range.start
            }
          }
          return this.monthDate
        },
        set: function (value) {
          this.monthDate = value
        }
      }
    }
  }
</script>
