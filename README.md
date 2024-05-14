# pg-notify-springboot-application
pg_notify-springboot-application  read event trigger in springboot application



CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL
);
CREATE OR REPLACE FUNCTION notify_all_sessions()
RETURNS TRIGGER AS $$
BEGIN
    PERFORM pg_notify('user_updates', row_to_json(NEW)::TEXT);
    RAISE NOTICE 'Notification sent for table %', TG_TABLE_NAME;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER users_notify_trigger
AFTER INSERT OR UPDATE OR DELETE ON users
FOR EACH ROW
EXECUTE PROCEDURE notify_all_sessions();

SELECT tgname, pg_get_triggerdef(oid, true) FROM pg_trigger WHERE tgrelid = 'users'::regclass;


INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');

=============================================================================================================================
maven dependecy
		<dependency>
			<groupId>org.postgresql</groupId>
			<artifactId>postgresql</artifactId>
			<version>42.2.23</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>

  ========================================================================================================================
  java class

package com.satellite.aunchers.config;

import org.postgresql.PGConnection;
import org.postgresql.PGNotification;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;
import org.springframework.transaction.event.TransactionalEventListener;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;

import jakarta.annotation.PostConstruct;

@Component
class PgNotificationListener {

	private final DataSource dataSource;

	public PgNotificationListener(DataSource dataSource) {
		this.dataSource = dataSource;
	}

	@PostConstruct
	public void setupListener() {
		new Thread(() -> {
			System.out.println("setupListener ====================");
			try (Connection connection = dataSource.getConnection()) {
				Statement statement = connection.createStatement();
				statement.execute("LISTEN user_updates");
				PGConnection pgConnection = connection.unwrap(PGConnection.class);
				while (true) {
					org.postgresql.PGNotification[] notifications = pgConnection.getNotifications();
					if (notifications != null) {
						for (PGNotification notification : notifications) {
							System.out.println("Received notification: " + notification.getName());
							System.out.println("Payload: " + notification.getParameter());
						}
					}
					Thread.sleep(1000); // adjust sleep time as needed
				}
			} catch (SQLException | InterruptedException e) {
				e.printStackTrace();
			}
		}).start();
	}
}
===============================================================================================================
output console
Received notification: user_updates
Payload: {"id":18,"name":"bhushan","email":"bbhus@gmnail.com"}

![image](https://github.com/bhushan0109/pg-notify-springboot-application/assets/75246941/253feda3-4ea3-48c2-8322-a978fa8bb754)

