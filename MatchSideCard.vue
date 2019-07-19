<template>
  <v-flex v-if="match && match.id">
    <v-card class="card-wrap">
      <v-card-title
        primary-title
        class="pb-2"
      >
        <v-flex class="v-headline mb-0 d-flex align-start xs6">
          {{ $t('common.mentorship') }}
        </v-flex>
        <v-flex class="d-flex justify-end xs6">
          <v-progress-linear
            class="my-0"
            :value="barWidth"
            height="8"
            color="secondary"
          />
        </v-flex>
        <v-flex class="subheadline d-flex justify-end align-center">
          {{ $t('session.lectures') }}
          {{ match.completedLection === match.program.totalLections ? match.completedLection : match.completedLection +
            1 }} /
          {{ match.program.totalLections }}
        </v-flex>
      </v-card-title>

      <v-card-text>
        <v-flex>
          <vv-avatars
            v-if="match.matchedParticipations"
            class="hidden-xs-only"
            :participation="match.matchedParticipations"
            size="60"
          >
            <div
              v-if="match"
              slot="data"
              class="data-bottom"
            >
              {{ firstUser.firstName }} & {{ secondUser.firstName }}
            </div>
          </vv-avatars>
        </v-flex>

        <v-flex
          v-if="getLatestAppointment(match)"
          class="mt-2"
        >
          <div class="content-time">
            {{ getLatestAppointment(match) | shortDate(true) }}
          </div>
          <div class="content-text">
            {{ $t('session.nextAppointment') }}
          </div>
        </v-flex>
        <v-flex v-else>
          <div class="content-text">
            {{ $t('activities.appointment.last') }}
          </div>
        </v-flex>

        <v-flex class="mt-3">
          <v-layout class="text-center wrap">
            <v-flex xs12>
              <div>
                <p class="meet">We meet {{match.matchType}}<v-icon @click="ModelMeet = true">edit</v-icon></p>
              </div>
            </v-flex>
          </v-layout>
        </v-flex>
        <v-dialog v-model="ModelMeet" max-width="500px">
          <v-card>
            <v-card-title>
              <h1
                v-t="'match.status.active.headline'"
                class="headline"
              />
              <span class="text" v-if="getLatestAppointment(match)">
                {{ $t('match.status.active.appointmentscheduled') }} {{ getLatestAppointment(match) | shortDate(true) }}
              </span>
              <span class="text" v-else>
                {{ $t('match.status.active.pleasePlanAppointment') }}
              </span>
            </v-card-title>
            <v-card-text>
              <div class="card-hint">
                <div class="text">
                  {{ $t('match.meeting.title') }}
                </div>
                <v-select
                  v-model="meetingTypeChoosen"
                  :items="meetingTypes"
                  class="dropdown ml-2"
                  @change="changeTypeOfMeeting($event)"
                />
              </div>
            </v-card-text>
            <v-card-actions>
              <v-btn color="primary" flat @click="ModelMeet=false">Close</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <v-flex class="mt-3">
          <v-layout class="text-center wrap">
            <v-flex xs12>
              <vv-base-button-arrow
                v-if="match.state === 'ACTIVE' && match.currentEnrollment.role === 'mentor' && match.completedLection > 3"
                color="primary"
                class="btn-card-arrow"
                :name="$t('match.endmentorship.btn')"
                @click="endMentorship = true"
              />
            </v-flex>
          </v-layout>
        </v-flex>
      </v-card-text>
    </v-card>

    <v-dialog
      v-model="endMentorship"
      :persistent="endMentorshipSubmitted"
      width="500"
    >
      <v-card>
        <v-card-title class="headline">
          <v-layout class="row">
            <v-flex
              d-flex
              class="align-center"
            >
              <h1 class="v-headline">
                {{ $t('match.endmentorship.title') }}
              </h1>
            </v-flex>

            <div class="closeBtn">
              <v-btn
                class="non-resize"
                flat
                icon
                @click="hideEndMentorshipModal()"
              >
                <v-icon>clear</v-icon>
              </v-btn>
            </div>
          </v-layout>
        </v-card-title>

        <v-card-text>
          <div v-if="endMentorshipSubmitted">
            <p v-if="endMentorshipWasSuccessful">
              {{ $t('match.endmentorship.success') }}
            </p>
            <p v-if="!endMentorshipWasSuccessful">
              {{ $t('match.endmentorship.failed') }}
            </p>
          </div>
          <div v-if="!endMentorshipSubmitted">
            <p>{{ $t('match.endmentorship.text') }}</p>

            <v-checkbox
              v-model="endMentorshipWasSuccessful"
              :label="$t('match.endmentorship.wasSuccessful')"
            />
            <v-expand-transition>
              <div v-if="!endMentorshipWasSuccessful">
                <v-textarea
                  v-model="endMsg"

                  class="mt-3"
                  :label="$t('match.endmentorship.feedback')"
                />
              </div>
            </v-expand-transition>


            <p>{{ $t('match.endmentorship.doYouWantToEnd') }}</p>
          </div>
        </v-card-text>
        <v-card-actions>
          <v-spacer />

          <div v-if="!endMentorshipSubmitted">
            <vv-base-button
              class="mr-2"
              color="grey"
              outline
              round
              @click="hideEndMentorshipModal()"
            >
              {{ $t('common.labels.back') }}
            </vv-base-button>
            <vv-base-button
              round
              color="primary"
              @click="submitEndMentorship()"
            >
              {{ $t('match.endmentorship.finalize') }}
            </vv-base-button>
          </div>
          <div v-else>
            <vv-base-button
              round
              color="primary"
              @click="hideEndMentorshipModal()"
            >
              {{ $t('common.close') }}
            </vv-base-button>
          </div>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-flex>
</template>
<script>
import { mapActions, mapGetters } from 'vuex';
  import { shortDate } from '@/shared/filters/DateFilter'; // eslint-disable-line
import getLatestMixin from '@/shared/mixins/getLatestMixin';

import VvSimpleCard from '@/shared/components/SimpleCard';
import VvBaseButton from '@/shared/components/BaseButton';
import VvBaseButtonArrow from '@/shared/components/BaseButtonArrow';
import VvAvatars from '@/shared/components/ParticipationAvatars';
import VvAvatar from '@/shared/components/Avatar';
import { showFirework, hideFirework } from '@/shared/components/fireworks';

export default {
  name: 'MatchSideCard',
  components: {
    VvSimpleCard,
    VvAvatars,
    VvAvatar,
    VvBaseButton,
    VvBaseButtonArrow,
  },
  mixins: [getLatestMixin],
  data() {
    return {
      isActived: true,
      callWindow: false,
      endMentorship: false,
      endMentorshipSubmitted: false,
      endMentorshipWasSuccessful: true,
      endMsg: '',
      ModelMeet: false,
      meetingTypeChoosen: '',
    };
  },
  computed: {
    ...mapGetters('global/user', {
      currentUser: 'getUser',
    }),
    ...mapGetters('mentoring/match', {
      match: 'getMatch',
    }),
    ...mapGetters('global/callchannel', {
      getConferenceLink: 'getConferenceLink',
    }),
    // firstUser aka CurrentUser
    firstUser() {
      return (this.match.matchedParticipations[0] && this.match.matchedParticipations[0].user)
        ? this.match.matchedParticipations[0].user : {};
      // const user = this.match.matchedParticipations.find(val => val.user.id === this.currentUser.id);
      // return user ? user.user : {};
    },
    // secondUser opponent
    secondUser() {
      return (this.match.matchedParticipations[1]
        && this.match.matchedParticipations[1].user)
        ? this.match.matchedParticipations[1].user : {};
    },
    otherUser() {
      const user = this.match.matchedParticipations.find(val => val.user.id !== this.currentUser.id);
      return user ? user.user : {};
    },
    barWidth() {
      const corrent = this.match.completedLection;

      return (
        // need to check if completed 10 will be 11/10
        this.match ? ((corrent + 1) / this.match.program.totalLections) * 100 : 0
      );
    },
    meetingTypes() {
      return [
        { text: this.$t('match.meeting.type.CLASSROOM'), value: 'CLASSROOM' },
        { text: this.$t('match.meeting.type.INPERSON'), value: 'INPERSON' },
        { text: this.$t('match.meeting.type.OTHERTOOL'), value: 'OTHERTOOL' },
      ];
    },
    // typeContainer() {
    //   this.$emit('type', {
    //     text: this.meetingTypeChoosen,
    //   });
    //   // console.log(this.meetingTypeChoosen);
    // },
  },
  methods: {
    ...mapActions('mentoring/match', [
      // 'sendMessage',
      'successfulMentorship',
      'unsuccessfulMentorship',
    ]),
    submitEndMentorship() {
      if (this.endMentorshipWasSuccessful) {
        this.successfulMentorship();
        try {
          showFirework();
        } catch (e) {
          console.error('fireworks', e);
        }
      } else {
        this.cancelMatch();
      }
      this.endMentorshipSubmitted = true;
    },

    cancelMatch() {
      try {
        this.unsuccessfulMentorship({ comment: this.endMsg });
      } catch (e) {
          console.log(e); //eslint-disable-line
      }
    },
    hideEndMentorshipModal() {
      hideFirework();
      this.endMentorship = false;
    },
    changeTypeOfMeeting(type) {
      this.$store.dispatch('mentoring/match/setTypeOfMeeting', { id: this.match.id, type });
    },
  },
  beforeDestroy() {
    // unset flames
    this.hideEndMentorshipModal();
  },
  // mounted() {
  //   this.$emit('type', {
  //     text: this.meetingTypeChoosen,
  //   });
  // },
};

</script>
<style lang="scss" scoped>
  @import '~@/styles/index';

  .v-card.card-wrap {
    min-width: 250px;
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.07);

    .v-headline {
      font-size: 12px !important;
      line-height: 18px !important;
      font-weight: 600;
      color: $primaryDark;
      text-transform: uppercase;
    }

    .link {
      cursor: pointer;
      transition: .2s;

      &:hover {
        color: $primary;
      }
    }

    .v-progress-linear {
      border-radius: 5px;
    }

    .subheadline {
      font-size: 12px;
      color: $secondaryGrey;
      line-height: 15px;
    }

    .content {
      &-time {
        font-size: 14px;
        font-weight: 600;
        text-align: center;
      }

      &-text {
        font-size: 12px;
        font-weight: 400;
        color: $primaryGrey;
        text-align: center;
      }
    }
  }

  .text-center {
    text-align: center
  }
  .meet {
    color: $secondaryGrey;
    font-size: 14px;
    .v-icon {
      font-size: 16px;
      color: $secondaryGrey;
      cursor: pointer;
    }
  }
  .card-hint {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .text, .headline {
    line-height: 1.57;
    color: $primaryGrey;
  }

  .non-resize {
    flex-shrink: 0 !important;
    flex-grow: 0 !important;
  }

  .closeBtn {
    position: absolute;
    top: 8px;
    right: 8px;
  }

</style>
