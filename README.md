# fcmmsg
FCM Test APP

Set up Firebase Cloud Messaging

Firebase Cloud Messaging lets you receive and send messages from your app to the server and other clients.
Firebase Cloud Messaging을 사용하면 앱에서 서버 및 다른 클라이언트로 메시지를 주고받을 수 있습니다.

This tutorial explains how to set up FCM and enable your app to receive notifications.
이 튜토리얼에서는 FCM을 설정하고 앱에서 알림을 수신하도록 설정하는 방법을 설명합니다.

https://firebase.google.com/docs/cloud-messaging/android/first-message

1. Connect your app to Firebase

2. Add FCM to your app

NOTE: After adding the SDK, here are some other helpful configurations to consider:
참고: SDK를 추가한 후 고려해야 할 몇 가지 유용한 구성은 다음과 같습니다.

* Do you want an easier way to manage library versions?
라이브러리 버전을 관리하는 더 쉬운 방법을 원하십니까?

You can use the Firebase Android BoM to manage your Firebase library versions and ensure that your app is always using compatible library versions.
Firebase Android BoM을 사용하여 Firebase 라이브러리 버전을 관리하고 앱이 항상 호환되는 라이브러리 버전을 사용하도록 할 수 있습니다.


3. Access the device registration token
장치 등록 토큰에 액세스

On initial startup of your app, the FCM SDK generates a registration token for the client app instance. 
앱을 처음 시작할 때 FCM SDK는 클라이언트 앱 인스턴스에 대한 등록 토큰을 생성합니다.

If you want to target single devices, or create device groups, you'll need to access this token.
단일 장치를 대상으로 하거나 장치 그룹을 만들려면 이 토큰에 액세스해야 합니다.


You can access the token's value by creating a new class which extends FirebaseInstanceIdService . 
FirebaseInstanceIdService 를 확장하는 새 클래스를 만들어 토큰 값에 액세스할 수 있습니다.

In that class, call getToken within onTokenRefresh , and log the value as shown:
해당 클래스에서 onTokenRefresh 내에서 getToken을 호출하고 다음과 같이 값을 기록합니다.

	@Override
	public void onTokenRefresh() {
		// Get updated InstanceID token.
		String refreshedToken = FirebaseInstanceId.getInstance().getToken();
		Log.d(TAG, "Refreshed token: " + refreshedToken);

		// If you want to send messages to this application instance or
		// manage this apps subscriptions on the server side, send the
		// Instance ID token to your app server.
		sendRegistrationToServer(refreshedToken);
	}

Also add the service to your manifest file:
또한 매니페스트 파일에 서비스를 추가합니다.

	<service
            android:name=".MyFirebaseInstanceIDService">
            <intent-filter>
            <action android:name="com.google.firebase.INSTANCE_ID_EVENT"/>
            </intent-filter>
        </service>

The onTokenRefresh callback fires whenever a new token is generated, so calling getToken in its context ensures that you are accessing a current, available registration token.
onTokenRefresh 콜백은 새 토큰이 생성될 때마다 실행되므로 해당 컨텍스트에서 getToken을 호출하면 현재 사용 가능한 등록 토큰에 액세스하고 있는지 확인할 수 있습니다.

FirebaseInstanceID.getToken() returns null if the token has not yet been generated.
FirebaseInstanceID.getToken()은 토큰이 아직 생성되지 않은 경우 null을 반환합니다.

After you have obtained the token, you can send it to your app server. See the Instance ID API reference for full detail on the API.
토큰을 얻은 후 앱 서버로 보낼 수 있습니다. API에 대한 자세한 내용은 인스턴스 ID API 참조를 확인하세요.

Instance ID API reference
(https://firebase.google.com/docs/reference/android/com/google/firebase/iid/FirebaseInstanceId?utm_source=studio)


4. Handle messages
메시지 처리

If you wish to do any message handling beyond receiving notifications on apps in the background, create a new Service ( File > New > Service > Service ) that extends FirebaseMessagingService. 
백그라운드에서 앱에 대한 알림 수신 이외의 메시지 처리를 수행하려면 FirebaseMessagingService 를 확장하는 새 서비스( File > New > Service > Service )를 만듭니다.

This service is necessary to receive notifications in foregrounded apps, to receive data payload, to send upstream messages, and so on.
이 서비스는 포그라운드 앱에서 알림 수신, 데이터 페이로드 수신, 업스트림 메시지 전송 등에 필요합니다.

In this service create an onMessageReceived method to handle incoming messages.
이 서비스에서 들어오는 메시지를 처리할 onMessageReceived 메서드를 만듭니다.

	@Override
        public void onMessageReceived(RemoteMessage remoteMessage) {
            // ...

            // TODO(developer): Handle FCM messages here.
            // Not getting messages here? See why this may be: https://goo.gl/39bRNJ
            Log.d(TAG, "From: " + remoteMessage.getFrom());

            // Check if message contains a data payload.
            if (remoteMessage.getData().size() > 0) {
		Log.d(TAG, "Message data payload: " + remoteMessage.getData());

		if (/* Check if data needs to be processed by long running job */ true) {
			// For long-running tasks (10 seconds or more) use Firebase Job Dispatcher.
			scheduleJob();
		} else {
			// Handle message within 10 seconds
			handleNow();
		}

            }

            // Check if message contains a notification payload.
            if (remoteMessage.getNotification() != null) {
		Log.d(TAG, "Message Notification Body: " + remoteMessage.getNotification().getBody());
            }

            // Also if you intend on generating your own notifications as a result of a received FCM
            // message, here is where that should be initiated. See sendNotification method below.
        }


Declare the following in your application's manifest:
애플리케이션의 매니페스트에서 다음을 선언합니다.

	<service
            android:name=".MyFirebaseMessagingService">
            <intent-filter>
            <action android:name="com.google.firebase.MESSAGING_EVENT"/>
            </intent-filter>
        </service>

If FCM is critical to the Android application's function, be sure to set android:minSdkVersion="8" or higher in the manifest. 
FCM이 Android 애플리케이션의 기능에 중요한 경우 매니페스트에서 android:minSdkVersion="8" 이상을 설정해야 합니다.

This ensures that the Android application cannot be installed in an environment in which it could not run properly.
이렇게 하면 Android 애플리케이션이 제대로 실행되지 않는 환경에 설치할 수 없습니다.


5. Next Steps

Once the client app is set up, you are ready to start sending downstream messages with Firebase console .
클라이언트 앱이 설정되면 Firebase 콘솔을 사용하여 다운스트림 메시지를 보낼 준비가 된 것입니다.

To add other, more advanced behavior to your app, you can declare an intent filter and implement an activity to respond to incoming messages.
다른 고급 동작을 앱에 추가하려면 인텐트 필터를 선언하고 수신 메시지에 응답하는 활동을 구현할 수 있습니다.

Send topic messages
https://firebase.google.com/docs/cloud-messaging/android/topic-messaging?utm_source=studio

Send to device groups
https://firebase.google.com/docs/cloud-messaging/android/device-group?utm_source=studio

Send upstream messages
https://firebase.google.com/docs/cloud-messaging/android/upstream?utm_source=studio

## 참고 사이트
Android 앱 프로젝트에 FCM설정 및 코드작성	
https://herojoon-dev.tistory.com/18

Android [Service took too long to process intent: com.google.android.c2dm.intent.RECEIVE App may get closed.]해결하기	
https://herojoon-dev.tistory.com/27
