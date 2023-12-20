---
layout: post
title: "My Journey with Spring Boot and my Warcraft ToDoList"
---

### About myself
Hello and welcome to my personal blog! I'm Martin, 31 years old, and here I aim to share my experiences with Spring Boot Security and OAuth2. Alongside insights into my achievements and challenges, you'll also get to learn about my recent completion of a 10-month Java Developer course at [everyone codes](https://everyonecodes.io/).

### Requirements for My Project "WoWToDoList" with API Integration

- Learn about OAuth2, including the Code flow and token generation.
- Successfully implement API endpoint connection to retrieve my characters' details from the Blizzard API.
- Develop an efficient database storage system for character details obtained through the API.
- Implement a ToDoList feature for each character with CRUD functionalities.
- Establish a one-to-many relationship between characters and their respective ToDoLists using annotations.


### The Problem I had with OAuth2 and generating the token

I began by creating my data classes, `Character` and `Task`, along with their respective repositories managed by Hibernate. I also established the necessary Service and Controller layers for these classes and added the Spring Security dependency into my pom.xml.

To start the OAuth2 integration, I added the required data such as clientID, SecretID, redirecturi, issueruri, etc. to my application.properties file. This laid the foundation for the later generation of a token.

The issue I quickly identified was that I was using the wrong method from Spring Security to create a token. I had implemented a method similar to the following pseudocode:

```java
public Map<String, Object> authentication(Authentication authentication) {
    return authentication.getPrincipal().getAttributes();
}
```
In the provided code, after a successful run, I only received user information and not a token that could be used for subsequent requests by adding it to my header as Bearer. Initially, I thought I needed to decode the received information using JWT, but that turned out not to be the case. I later discovered the correct solution.


### The Solution

**Security Configuration for OAuth2 Integration**

This snippet represents a security configuration using Spring Security to integrate OAuth2. It disables CSRF protection, configures OAuth2 login, and sets the default success URL to "/character/profile". The authorization endpoint is specified with a base URI of "/oauth2/authorization".

```java
@Override
SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        
        http
            .csrf(AbstractHttpConfigurer::disable)
            .oauth2Login(oauth2login -> oauth2login
                .defaultSuccessUrl("/character/profile")
                .authorizationEndpoint(authorizationEndpointConfig -> authorizationEndpointConfig
                    .baseUri("/oauth2/authorization")));
        
        return http.build();
````


**Generating the Token**

This method is crucial for obtaining the access token from the authentication object, facilitating access to protected resources on behalf of the authenticated user. 
```java
protected String getAccessToken(Authentication authentication) {

    OAuth2AuthenticationToken oauthToken = (OAuth2AuthenticationToken) authentication;
    OAuth2AuthorizedClient authorizedClient = authorizedClientService
        .loadAuthorizedClient(oauthToken.getAuthorizedClientRegistrationId(), oauthToken.getName());
        
    return authorizedClient.getAccessToken().getTokenValue();
}
````

### Retrieving and Processing Data from the Blizzard API

After successfully obtaining the access token, I could finally send API requests to the Blizzard API. Upon inserting the required headers and parameters and receiving the initial data, I used the Jackson Library and ObjectMapper to extract the name and server of the character. Then, I performed a duplicate check using Hibernate before rendering the extracted character information in the frontend with Thymeleaf. Additionally, using JavaScript to fetch the data and send it to my backend where I stored the character in the database.

This process involved the following steps:

1. **Sending API Requests:**
   - Utilized the acquired access token to authenticate API requests to the Blizzard API.

2. **Data Extraction with Jackson and ObjectMapper:**
   - Employed the Jackson Library and ObjectMapper to parse the received data and extract relevant information, such as the character's name and server.

3. **Duplicate Check with ORM:**
   - Implemented a duplicate check using the ORM functionality in the warcraftCharacter repository with the method:
     ```java
     Optional<WarcraftCharacter> findByNameAndServer(String name, String server);
     ```
     This method allowed me to implement a simple logic ensuring that if the character is already in the database, it should not be displayed.

4. **Frontend Display with Thymeleaf:**
   - Integrated Thymeleaf to render the extracted character information in the frontend.

5. **Database Storage with JavaScript fetch to backend:**
   - Used JavaScript to fetch and send the information to my backend to store the character details in the database.

These steps collectively allowed for the seamless integration of data retrieval, processing, and presentation within the web application, with the added benefit of preventing duplicate entries in the frontend based on a logical check.
