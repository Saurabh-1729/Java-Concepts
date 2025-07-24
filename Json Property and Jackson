Let's understand @JsonProperty with a real-world example!

Think of @JsonProperty like a translator between two languages - JSON and Java.

// JSON coming from an API:
{
    "user_first_name": "John",
    "user_age": 25,
    "is_active": true
}

// Our Java class using @JsonProperty:
@Data
public class User {
    @JsonProperty("user_first_name")
    private String firstName;
    
    @JsonProperty("user_age")
    private int age;
    
    @JsonProperty("is_active")
    private boolean active;
}

Real-world analogy: Imagine you're at an international airport with signs in multiple languages:

Airport sign says "Baggage Claim" in English
Same sign says "Réclamation des bagages" in French
Both point to the same place
Similarly, @JsonProperty works like these multilingual signs:

JSON says "user_first_name"
Java says "firstName"
@JsonProperty tells Jackson "these mean the same thing"
When you use the code:

ObjectMapper mapper = new ObjectMapper();
String json = "{'user_first_name': 'John'}";
User user = mapper.readValue(json, User.class);
System.out.println(user.getFirstName()); // Prints: John

Jackson sees @JsonProperty and knows:

"user_first_name" in JSON → maps to → firstName in Java
"user_age" in JSON → maps to → age in Java
"is_active" in JSON → maps to → active in Java
This makes working with different naming conventions between JSON APIs and Java code smooth and automatic!
