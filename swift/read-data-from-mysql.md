# How To Display Data From a MySQL database into a SwiftUI app using PHP

## Getting Started

To display data from a MySQL database in a SwiftUI app using PHP, you'll need to follow these steps:

- **Set up the MySQL database** - Ensure your MySQL database and tables are set up.
- **Create a PHP script to fetch data from the database** - This script will query the database and return the data in a JSON format.
- **Create a network request from SwiftUI** - Use URLSession to fetch data from the PHP script.
- **Parse the JSON data and display it in the SwiftUI view**


## Set Up the MySQL Database
Assume you have a MySQL database named my_database and a table named users with the following schema:

```
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


## Create a PHP Script to Fetch Data
Create a PHP script named fetch_users.php to query the database and return the data in JSON format.

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

// Fetch data
$sql = "SELECT id, name, email, created_at FROM users";
$result = $conn->query($sql);

$users = array();

if ($result->num_rows > 0) {
    while($row = $result->fetch_assoc()) {
        $users[] = $row;
    }
}

echo json_encode($users);

$conn->close();
?>
```


## Create a Network Request from SwiftUI

In your SwiftUI app, create a function to fetch the data from the PHP script using URLSession.

### Models and ViewModel

- First, create a model to represent the user data and a ViewModel to manage fetching data.

```
import SwiftUI
import Combine

// User model
struct User: Codable, Identifiable {
    let id: Int
    let name: String
    let email: String
    let created_at: String
}

// ViewModel to handle fetching data
class UserViewModel: ObservableObject {
    @Published var users: [User] = []
    
    func fetchUsers() {
        guard let url = URL(string: "https://yourserver.com/fetch_users.php") else { return }
        
        URLSession.shared.dataTask(with: url) { data, response, error in
            guard let data = data else {
                print("No data in response: \(error?.localizedDescription ?? "Unknown error").")
                return
            }
            
            if let decodedResponse = try? JSONDecoder().decode([User].self, from: data) {
                DispatchQueue.main.async {
                    self.users = decodedResponse
                }
            } else {
                print("Invalid response from server.")
            }
        }.resume()
    }
}
```

### SwiftUI View

- Now, create a SwiftUI view to display the fetched data.


```
import SwiftUI

struct ContentView: View {
    @StateObject private var viewModel = UserViewModel()
    
    var body: some View {
        NavigationStack {
            List(viewModel.users) { user in
                VStack(alignment: .leading) {
                    Text(user.name)
                        .font(.headline)
                    Text(user.email)
                        .font(.subheadline)
                }
            }
            .navigationTitle("Users")
            .onAppear {
                viewModel.fetchUsers()
            }
        }
    }
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


## Explanation

- **PHP Script (fetch_users.php)**
    - **Database Connection** - Establishes a connection to the MySQL database.
    - **Data Fetching** - Executes a SQL query to fetch all user records.
    - **JSON Encoding** - Encodes the fetched data as a JSON array and prints it.
- **SwiftUI App**
    - **Model** - Defines a User struct that conforms to Codable and Identifiable.
    - **ViewModel** - Defines a UserViewModel class that fetches data from the PHP script and publishes it to the view.
    - **Networking** - Uses URLSession to fetch data from the PHP script and decodes the JSON response.
    - **View** - A List view displays the fetched user data. The onAppear modifier calls the fetchUsers function when the view appears.
- **Networking and JSON Decoding**
    - **URLSession** - Configures a GET request to fetch data from the PHP script.
JSONDecoder: Decodes the JSON response into an array of User objects.
DispatchQueue: Updates the published users property on the main thread to ensure UI updates are performed correctly.
Replace "https://yourserver.com/fetch_users.php" with the actual URL where your PHP script is hosted. This setup ensures your SwiftUI app can fetch and display data from a MySQL database using a PHP backend.
