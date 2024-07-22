# Saving Data to MySQL Database

## Overview

This example will use PHP as the scripting language that'll work between SwiftUI and a MySQL database. 

What are some alternatives?

- **CloudKit** - While CloudKit is a great option for iOS developers, it's not an option if you hope to one day have an Android version of your app.
- **Firebase** - A great option with somewhat of a low barrier to entry. All of the features that Firebase provides could be overwhelming if you just want a simple database and cost can be a factor if your app grows.
- **AWS Amplify** - I've also explored Amplify as an option. Even more so than Firebase, the amount of features that Amplify provides can both be a plus and minus. All those features can be overwhelming if you just want a simple database.



## Getting Started

Saving data from a SwiftUI app to a MYSQL database using PHP involves the following steps:

- Set up the MySQL database
- Create a PHP script
- Make a network request from SwiftUI


## Set up the MySQL database

1. Create a database with a name of `myDatabase` 
2. Create a table named `users` within the `myDatabase` database.

Example Table setup:
```
CREATE TABLE users {
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL, 
    email VARCHAR(255) NOT NULL,
    created_at TIMESTAMP Default CURRENT_TIMESTAMP 
};

```

## Create a PHP Script

The PHP script will handle the incoming data from the SwiftUI app and insert it into the MySQL database.

- Create a PHP script named `save_user.php` that will handle the data and insert it into the database.

```
<?php
$servername = "localhost";
$username = "your_username";
$password = "your_password";
$dbname = "my_database";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

// Get the posted data
$name = $_POST['name'];
$email = $_POST['email'];

// Prepare and bind
$stmt = $conn->prepare("INSERT INTO users (name, email) VALUES (?, ?)");
$stmt->bind_param("ss", $name, $email);

if ($stmt->execute()) {
    echo json_encode(["success" => true]);
} else {
    echo json_encode(["success" => false, "error" => $stmt->error]);
}

$stmt->close();
$conn->close();
?>
```

## Make a Network Request from SwiftUI

With the database and PHP script set up, you can now, in your SwiftUI app, create a function to send data to the PHP script using `URLSession`.

```
import SwiftUI

struct ContentView: View {
    @State private var name: String = ""
    @State private var email: String = ""
    @State private var message: String = ""
    
    var body: some View {
        NavigationStack {
            Form {
                Section(header: Text("User Information")) {
                    TextField("Name", text: $name)
                    TextField("Email", text: $email)
                }
                
                Section {
                    Button(action: submitForm) {
                        Text("Submit")
                    }
                }
                
                Section {
                    Text(message)
                }
            }
            .navigationTitle("User Form")
        }
    }
    
    private func submitForm() {
        guard let url = URL(string: "https://yourserver.com/save_user.php") else { return }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        
        let bodyData = "name=\(name)&email=\(email)"
        request.httpBody = bodyData.data(using: .utf8)
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            guard let data = data else {
                print("No data in response: \(error?.localizedDescription ?? "Unknown error").")
                return
            }
            
            if let decodedResponse = try? JSONDecoder().decode(ServerResponse.self, from: data) {
                DispatchQueue.main.async {
                    if decodedResponse.success {
                        self.message = "User saved successfully!"
                    } else {
                        self.message = "Failed to save user: \(decodedResponse.error ?? "Unknown error")"
                    }
                }
            } else {
                print("Invalid response from server.")
            }
        }.resume()
    }
}

struct ServerResponse: Codable {
    let success: Bool
    let error: String?
}

@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```


## Further Explanation

### PHP Script

The PHP Script (`save_user.php`) does the following:
- Database Connection: Establishes a connection to the MySQL database.
- Data Retrieval: Retrieves `name` and `email` from the POST request.
- SQL Injection Protection: Uses prepared statements to insert the data into the database.
- Response: Returns a JSON response indicating success or failure.

### SwiftUI App

- **State Variables**: Uses `@State` properties to manage the user's input and feedback message.
- **Form**: Collects user information (name and email) and includes a submit button.
- **Network Request**: Sends the collected data to the PHP script using URLSession.
- **JSON Decoding**: Decodes the JSON response from the PHP script to determine success or failure.

### Networking

- **URLSession**: Configures a POST request with the user's data.
- **Data Task**: Sends the request and handles the response, updating the UI accordingly.

This setup ensures that your SwiftUI app can send data to a server-side PHP script, which then saves the data to a MySQL database. 
Make sure to replace `https://yourserver.com/save_user.php` with the actual URL where your PHP script is hosted.

