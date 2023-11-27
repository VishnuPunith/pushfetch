<template>
  <div>
    <h1>Firebase Messaging Demo</h1>
    <div>
      <strong>Device Token:</strong> {{ token }}
    </div>
    <div>
      <strong>Received Message:</strong> {{ receivedMessage }}
    </div>
    <div>
      <strong>Notifications:</strong> {{ notifications }}
    </div>
    <div>
      <strong>Error:</strong> {{ error }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      token: '',
      receivedMessage: '',
      notifications: '',
      error: '',
    };
  },
  created() {
    this.setupFirebase();
    this.requestNotificationPermission();
    this.listenForMessages();
    this.fetchFirestoreData();
  },
  methods: {
    setupFirebase() {
      // TODO: Replace with your Firebase project's configuration
      const firebaseConfig = {
        apiKey: "<YOUR_API_KEY>",
        authDomain: "<YOUR_AUTH_DOMAIN>",
        projectId: "<YOUR_PROJECT_ID>",
        messagingSenderId: "<YOUR_MESSAGING_SENDER_ID>",
        appId: "<YOUR_APP_ID>",
        measurementId: "<YOUR_MEASUREMENT_ID>",
      };
      firebase.initializeApp(firebaseConfig);
      this.messaging = firebase.messaging();
    },
    requestNotificationPermission() {
      if (Notification.permission === 'default') {
        Notification.requestPermission()
          .then((permission) => {
            if (permission === 'granted') {
              return this.messaging.getToken({
                vapidKey: "<YOUR_VAPID_KEY>",
              });
            }
          })
          .then((token) => {
            this.token = token;
          })
          .catch((err) => {
            this.error = `Unable to get permission to notify: ${err}`;
          });
      }
    },
    listenForMessages() {
      this.messaging.onMessage((payload) => {
        this.receivedMessage = JSON.stringify(payload);
        const enableForegroundNotification = true;
        if (enableForegroundNotification) {
          const notification = payload.notification;
          if ('serviceWorker' in navigator) {
            navigator.serviceWorker.getRegistration()
              .then((registration) => {
                registration.showNotification(notification.title);
              });
          }
        }
      });
    },
    fetchFirestoreData() {
      const firebaseConfig = {/* Your Firebase config here */};
      const app = firebase.initializeApp(firebaseConfig);
      const firestore = app.firestore();
      const usersCollection = firestore.collection('users');
      usersCollection.get()
        .then((querySnapshot) => {
          const data = [];
          querySnapshot.forEach((doc) => {
            data.push({ id: doc.id, ...doc.data() });
          });
          this.notifications = JSON.stringify(data);
        })
        .catch((error) => {
          this.error = `Error fetching documents: ${error}`;
        });
    },
  },
};
</script>
