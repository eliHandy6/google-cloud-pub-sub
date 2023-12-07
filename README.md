

student-02-a4f01333bfa9@qwiklabs.net

odvehVjunRQd

qwiklabs-gcp-03-2a0583235bff

Task 1. Fetch the application source files:

	In Cloud Shell, enter the following command to create an environment variable that contains the project ID for this lab:
	export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
	
	Verify that the demo application files were created:
	It may take a few minutes for provisioning to complete and the bucket to be created.
	gcloud storage ls gs://$PROJECT_ID
	
	

	Copy the application folders to Cloud Shell:
	gcloud storage cp -r gs://$PROJECT_ID/* ~/

	Make the Maven wrapper scripts executable:
	chmod +x ~/guestbook-frontend/mvnw
	chmod +x ~/guestbook-service/mvnw

Task 2. Enable Pub/Sub API:
	gcloud services enable pubsub.googleapis.com

Task 3. Create a Pub/Sub topic:
	gcloud pubsub topics create messages

Task 4. Add Spring Cloud GCP Pub/Sub starter:

	in the code editor, open ~/guestbook-frontend/pom.xml.
	   
	Insert the following new dependency at the end of the <dependencies> section, just before the closing </dependencies> tag:
	
		<dependency>
		    <groupId>org.springframework.cloud</groupId>
		    <artifactId>spring-cloud-gcp-starter-pubsub</artifactId>
		</dependency>
		
Task 5. Publish a message:

	Open guestbook-frontend/src/main/java/com/example/frontend/FrontendController.java in the Cloud Shell code editor.:
	Add the following statement immediately after the existing import directives:
	
		    import org.springframework.cloud.gcp.pubsub.core.*
	
	Insert the following statement between the lines private GuestbookMessagesClient client; and @Value("${greeting:Hello}")
	
		    @Autowired
		    private PubSubTemplate pubSubTemplate;

	Add the following statement inside the if statement to process messages that aren't null or empty, just below the comment // Post the message to the backend service:
	
             pubSubTemplate.publish("messages", name + ": " + message)

Task 6. Test the application in the Cloud Shell:

	In Cloud Shell, change to the guestbook-service directory:
	
	cd ~/guestbook-service
	
	Run the backend service application:
	
	./mvnw spring-boot:run -Dspring-boot.run.jvmArguments="-Dspring.profiles.active=cloud"
	The backend service application launches on port 8081.This takes a minute or two to complete. You should wait until you see that the GuestbookApplication is running.:


	Open a new Cloud Shell session tab to run the frontend application by clicking the plus (+) icon to the right of the title tab for the initial Cloud Shell session.:
	Change to the guestbook-frontend directory:
	cd ~/guestbook-frontend
	Copied!
	Start the frontend application with the cloud profile:
	./mvnw spring-boot:run -Dspring.profiles.active=cloud
	Copied!

Task 7. Create a subscription:

	 Pub/Sub supports pull subscription and push subscription. With a pull subscription, the client can pull messages from the topic. With a push subscription, Pub/Sub can publish messages to a target webhook endpoint.

	A topic can have multiple subscriptions. A subscription can have many subscribers. If you want to distribute different messages to different subscribers, then each subscriber needs to subscribe to its own subscription. If you want to publish the same messages to all the subscribers, then all the subscribers must subscribe to the same subscription.

	Pub/Sub messages are delivered "at least once." Thus, you must deal with idempotence and you must deduplicate messages if you cannot process the same message more than once.


	Create a Pub/Sub subscription:
	gcloud pubsub subscriptions create messages-subscription-1 \
	  --topic=messages
	
	Pull messages from the subscription:
	gcloud pubsub subscriptions pull messages-subscription-1
	
	The pull messages command should report 0 items.
	The message you posted earlier does not appear, because the message was published before the subscription was created.

	Return to the frontend application, post another message, and then pull the message again:
	gcloud pubsub subscriptions pull messages-subscription-1
	
	The message appears. The message remains in the subscription until it's acknowledged.

	Pull the message again and remove it from the subscription by using the auto-acknowledgement switch at the command line:
	gcloud pubsub subscriptions pull messages-subscription-1 --auto-ack




Task 8. Process messages in subscriptions:

	cd ~
	curl https://start.spring.io/starter.tgz \
	  -d type=maven-project \
	  -d dependencies=web,cloud-gcp-pubsub \
	  -d bootVersion=2.2.4RELEASE \
	  -d javaVersion=1.8 \
	  -d baseDir=message-processor | tar -xzvf -
	  
	 
	 To write the code to listen for new messages delivered to the topic, open ~/message-processor/src/main/java/com/example/demo/DemoApplication.java in the Cloud Shell code editor.
	Add the following import directives below the existing import directives:
	
	import org.springframework.context.annotation.Bean;
	import org.springframework.boot.ApplicationRunner;
	import com.google.cloud.spring.pubsub.core.*;


	    @Bean
	    public ApplicationRunner cli(PubSubTemplate pubSubTemplate) {
		return (args) -> {
		    pubSubTemplate.subscribe("messages-subscription-1",
		        (msg) -> {
		            System.out.println(msg.getPubsubMessage()
		                .getData().toStringUtf8());
		            msg.ack();
		        });
		};
	    }



	Add this line to change the port on message-processor/src/main/resources/application.properties:
	server.port=${PORT:9090}

	Return to the Cloud Shell tab for the message processor to listen to the topic:
	cd ~/message-processor
	./mvnw -q spring-boot:run
	
	
	Open the browser with the frontend application, and post a few messages.
	Verify that the Pub/Sub messages are received in the message processor.





	
