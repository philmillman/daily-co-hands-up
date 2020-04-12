<template>
  <div class="container">
    <div id="callframe-target">
    </div>
    <div class="content">
      <h3 v-show="!hasJoined">Click to join</h3>
      <br>
      <div class="controls-container">
        <button @click="joinCall" 
        title="Join Call" 
        :disabled="buttonsDisabled.join">📲</button>
        <button @click="leaveCall" 
        title="Leave Call" 
        :disabled="buttonsDisabled.leave">❌</button>
        <button @click="toggleLocalHandState"
        :title="handTitle"
        :disabled="buttonsDisabled.hand">{{ handEmoji }}</button>
      </div>
      <div class="participants-container" v-show="hasJoined">
        <h3>Participants</h3>
        <ul >
          <li v-for="(participant, index) of participants" :key="index" :data-user_id="participant.user_id">
            <span v-if="participantsHandsRaised.includes(participant.user_id)">
              🙌 
            </span>
            {{ getParticipantName(participant) }} - {{ getParticipantBadgeString(participant)}}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import DailyIframe from '@daily-co/daily-js';
const CALL_OPTIONS = {
  url: 'https://hangout-beer.daily.co/tutorial',
  iframeStyle: {
    position: 'fixed',
    top: 0,
    left: 0,
    width: '80%',
    height: '80%'
  }
}

export default {
  name: 'Call',
  data: function () {
    return {
      buttonsDisabled: {
        join: false,
        leave: true,
        hand: true
      },
      hasJoined: false,
      participants: [],
      participantsHandsRaised: [],
      localParticipant: {},
      handRaised: false,
      callframe: {}
    }
  },
  computed: {
    handTitle: function () {
      return this.handRaised ? "Lower Your Hand" : "Raise Your Hand";
    },
    handEmoji: function () {
      return this.handRaised ? "🙌" : "✋";
    }
  },
  mounted: function () {
    const targetEl = document.getElementById('callframe-target')
    this.callframe = DailyIframe.createFrame(
      targetEl,
      CALL_OPTIONS
    ); 

    this.callframe
      .on('joined-meeting', (e) => {
        console.log("Event: ",e);
        this.hasJoined = true;
      })
      .on('participant-joined', (e) => {
        console.log("Event: ",e);
        if(e.participant.user_id !== this.localParticipant.user_id) {
            this.updateParticipants();
            setTimeout(() => {
              this.shareHandState()
            }, 5000)
        }
      })
      .on('participant-updated', (e) => {
        console.log("Event: ",e);
        this.updateParticipants();
      })
      .on('participant-left', (e) => {
        console.log("Event: ",e);
        this.updateParticipants();
        this.updateRaisedHands({handState: false, ...e.participant});
      })
      .on('app-message', ({ data }) => {
        this.updateRaisedHands(data);
      });
  
  },
  methods: {
     joinCall: async function () {
      this.buttonsDisabled.join = true; 
      try {
        const participants = await this.callframe.join();
        this.participants = Object.keys(participants).map((key) => participants[key]);
        this.localParticipant = participants.local;
        this.buttonsDisabled.leave = false; 
        this.buttonsDisabled.hand = false; 
      } catch (e) {
        console.error('ERROR while joining meeting', e);
        return;
      }

    },
    getParticipantName: function (participant) {
      let name = "";
      if (participant.user_name) {
        name = participant.user_name;
      } else if (participant.owner) {
        name = "Host";
      } else {
        name = "guest"
      }

      if (participant.local) {
        name += " (you)";
      }
      return name;
    },
    getParticipantBadgeString: function (participant) {
      let badgeString = "";
      if (participant.owner) {
        badgeString += "🏠"
      }
      if (participant.audio) {
        badgeString += "🔈"
      }
      if (participant.video) {
        badgeString += "🎦"
      }

      return badgeString;
    },
    updateParticipants: async function () {
      const participants =  await this.callframe.participants();
      this.participants = Object.keys(participants).map((key) => participants[key]);
    },
    
    toggleLocalHandState: function () {
      this.handRaised = !this.handRaised;
      this.updateRaisedHands({handState:this.handRaised, ...this.localParticipant});
      this.shareHandState();

    },
    updateRaisedHands: function({ handState, user_id}) {
      if (!this.participantsHandsRaised.includes(user_id) && handState) {
        this.participantsHandsRaised.push(user_id);
      } else if (this.participantsHandsRaised.includes(user_id) && !handState) {
        this.participantsHandsRaised.splice(this.participantsHandsRaised.indexOf(user_id),1)
      }
    },
    shareHandState: function() {
      this.callframe.sendAppMessage(
        { handState : this.handRaised,
          ...this.localParticipant }
      )
    },
    leaveCall: function () {
      //reset initial state
      this.callframe.leave();
      this.participants = {};
      this.participantsHandsRaised = [];
      this.localParticipant = {};
      this.handRaised = false;
      this.buttonsDisabled.join = false; 
      this.buttonsDisabled.leave = true; 
      this.buttonsDisabled.hand = true; 
      this.hasJoined = false;
    },

  }

}
</script>

<style scoped>

.content {
  padding-left: 80%;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}

button:disabled {
  opacity: 0.25;
}
</style>
