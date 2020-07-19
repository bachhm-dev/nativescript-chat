<template>
  <Page>
    <ActionBar title="Firebase chat" />
    <StackLayout>
      <ScrollView height="90%" width="100%">
        <ListView for="msg in messages" class="message-item">
          <v-template>
            <GridLayout columns="*" rows="auto" class="msg">
              <StackLayout
                :class="filter(msg.from)"
                orientation="horizontal"
                :horizontalAlignment="align(msg.from)"
                paddingLeft="5"
              >
                <!-- <Label :class="'message-autor'" :text="msg.from" verticalAlignment="center"></Label> -->
                <Label
                  :class="'msg_text'"
                  :text="msg.message"
                  textWrap="true"
                  verticalAlignment="top"
                ></Label>
              </StackLayout>
            </GridLayout>
          </v-template>
        </ListView>
      </ScrollView>
      <StackLayout height="10%">
        <GridLayout columns="*,auto" style="padding: 10">
          <TextField
            v-model="message"
            class="chatTextField"
            row="0"
            col="0"
            @returnPress="saveMessage"
          ></TextField>
          <Button class="chatBtn" row="0" col="1" text="send" @tap="saveMessage"></Button>
        </GridLayout>
      </StackLayout>
    </StackLayout>
  </Page>
</template>
<script>
const firebase = require("nativescript-plugin-firebase/app");
firebase.initializeApp({
  persist: false
});

export default {
  data() {
    return {
      messages: [],
      message: undefined,
      username: "NAGORA"
    };
  },
  created() {
    this.fetchMessages();
  },
  methods: {
    saveMessage() {
      const vm = this;
      if (this.message) {
        firebase
          .firestore()
          .collection("chat")
          .add({
            message: this.message,
            createAt: new Date(),
            from: this.username
          })
          .then(function(docRef) {
            vm.message = null;
          })
          .catch(function(error) {
            console.error("Error adding document: ", error);
          });
      }
    },
    fetchMessages() {
      let vm = this;
      firebase
        .firestore()
        .collection("chat")
        .orderBy("createAt")
        .onSnapshot(querySnapshot => {
          const allMessages = [];
          querySnapshot.forEach(doc => {
            allMessages.push(doc.data());
          });
          vm.messages = allMessages;
        });
    },
    align(from) {
      if (from === this.username) {
        return "right";
      }
      return "left";
    },
    filter(from) {
      if (from === this.username) {
        return "me";
      } else {
        return "them";
      }
    }
  }
};
</script>
<style scoped>
ActionBar {
  background-color: #53ba82;
  color: #ffffff;
}
ListView {
  separator-color: white;
}

.msg {
  font-size: 14;
  padding: 5;
}

.me .msg_text {
  background-color: #30a9ff;
  color: white;
  padding: 7;
  margin-left: 10;
  border-radius: 5;
}

.them .msg_text {
  background-color: #e0e0e0;
  color: #333;
  padding: 7;
  border-radius: 5;
  /* margin-right: 40; */
}

.authorimg {
  margin: 0 5 5 5;
  width: 30;
  height: 30;
  border-radius: 15;
}

.chatTextField {
  padding: 5;
  background-color: lightgray;
  border-radius: 4;
}
</style>
