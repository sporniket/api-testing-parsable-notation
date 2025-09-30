# Booking a meeting room

This sample demonstrates a typical workflow for booking a meeting room using a fictitious API.

> TODO : require `moment.js` in globals and compute meeting date in 7 days


## :Preamble:

This preamble sets up global variables and defines common actions for our meeting room booking tests.

### :Globals:

Define base URL, API key, and initial booking details.

```js 
// Global setup for the meeting room booking API tests
const baseUrl = "https://api.meetings.example.com/v1";
const apiKey = "your_secure_api_key_here"; // In a real scenario, this would be more securely handled
let authToken = ""; // To be obtained via login
let selectedRoomId = "";
let bookingId = "";
let meetingTopic = "Project Alpha Sync";
let meetingDate = "2024-07-25";
let meetingStartTime = "10:00";
let meetingEndTime = "11:00";
let requiredPax = 5;
```

### :Before all:

This script runs once before any test step to ensure we have a clean slate or initial setup.

```js 
// Pre-test suite setup
console.log("Starting Meeting Room Booking API test suite.");
// Potentially clear any existing test data related to specific rooms or users
// Or ensure test user exists
```

### :After all:

This script runs once after all test steps have completed.

```js 
// Post-test suite cleanup
console.log("Meeting Room Booking API test suite finished.");
```

## :Step: 1. Authenticate and obtain an API token

This step simulates a user login to get an authentication token required for subsequent API calls.

```js 
// Pre-request script for authentication
let username = "testuser@example.com";
let password = "securepassword123";
```

```http
POST {{baseUrl}}/auth/login
Content-Type: application/json
Accept: application/json
X-API-Key: {{apiKey}}

{
    "email": "{{username}}",
    "password": "{{password}}"
}
```

```js 
// Post-request script to store the authentication token
if (response.status === 200) {
    const responseBody = JSON.parse(response.body);
    authToken = responseBody.token;
    console.log("Authentication successful. Token obtained.");
} else {
    throw new Error(`Authentication failed: Status ${response.status}, Body: ${response.body}`);
}
```

## :Step: 2. Search for available meeting rooms

This step searches for rooms that can accommodate the required number of participants on a specific date and time.

```js 
// Pre-request script for room search
console.log(`Searching for rooms for ${requiredPax} pax on ${meetingDate} from ${meetingStartTime} to ${meetingEndTime}`);
```

```http
GET {{baseUrl}}/rooms/available?date={{meetingDate}}&startTime={{meetingStartTime}}&endTime={{meetingEndTime}}&capacity={{requiredPax}}
Accept: application/json
Authorization: Bearer {{authToken}}
```

```js 
// Post-request script to process search results
if (response.status === 200) {
    const rooms = JSON.parse(response.body);
    if (rooms && rooms.length > 0) {
        // Select the first available room
        selectedRoomId = rooms[0].id;
        console.log(`Found and selected room: ${selectedRoomId} (${rooms[0].name})`);
    } else {
        throw new Error("No available rooms found for the specified criteria.");
    }
} else {
    throw new Error(`Room search failed: Status ${response.status}, Body: ${response.body}`);
}
```

## :Step: 3. Create a new meeting booking

With a room selected, this step attempts to create a new booking for that room.
```js
// Pre-request script for booking creation
if (!selectedRoomId) {
    throw new Error("No room was selected in the previous step.");
}
console.log(`Attempting to book room ${selectedRoomId} for topic: "${meetingTopic}"`);
let organizerEmail = "organizer@example.com";
let attendee1Email = "attendee1@example.com";
let attendee2Email = "attendee2@example.com";
```

```http
POST {{baseUrl}}/bookings
Content-Type: application/json
Accept: application/json
Authorization: Bearer {{authToken}}

{
    "roomId": "{{selectedRoomId}}",
    "date": "{{meetingDate}}",
    "startTime": "{{meetingStartTime}}",
    "endTime": "{{meetingEndTime}}",
    "topic": "{{meetingTopic}}",
    "organizer": {
        "email": "{{organizerEmail}}",
        "name": "Meeting Organizer"
    },
    "attendees": [
        {"email": "{{attendee1Email}}"},
        {"email": "{{attendee2Email}}"}
    ]
}
```

```js 
// Post-request script to verify the booking and store its ID
if (response.status === 201) {
    const bookingDetails = JSON.parse(response.body);
    bookingId = bookingDetails.id;
    console.log(`Booking created successfully with ID: ${bookingId}`);
} else {
    throw new Error(`Booking creation failed: Status ${response.status}, Body: ${response.body}`);
}
```

## :Step: 4. Retrieve booking details

This step fetches the details of the newly created booking to verify its data integrity.
```js 
// Pre-request script for retrieving booking details
if (!bookingId) {
    throw new Error("No booking ID available from the previous step.");
}
console.log(`Retrieving details for booking ID: ${bookingId}`);
```

```http
GET {{baseUrl}}/bookings/{{bookingId}}
Accept: application/json
Authorization: Bearer {{authToken}}
```

```js // Post-request script to validate retrieved booking details
if (response.status === 200) {
    const retrievedBooking = JSON.parse(response.body);
    if (retrievedBooking.id !== bookingId) {
        throw new Error(`Retrieved booking ID mismatch: Expected ${bookingId}, got ${retrievedBooking.id}`);
    }
    if (retrievedBooking.topic !== meetingTopic) {
        throw new Error(`Retrieved booking topic mismatch: Expected "${meetingTopic}", got "${retrievedBooking.topic}"`);
    }
    console.log(`Booking details retrieved and validated for ID: ${bookingId}. Topic: "${retrievedBooking.topic}"`);
} else {
    throw new Error(`Failed to retrieve booking details: Status ${response.status}, Body: ${response.body}`);
}
```

## :Step: 5. Cancel the created booking

This step cleans up the test by cancelling the meeting room booking.

```js // Pre-request script for canceling the booking
if (!bookingId) {
    throw new Error("No booking ID available to cancel.");
}
console.log(`Attempting to cancel booking ID: ${bookingId}`);
```

```http
DELETE {{baseUrl}}/bookings/{{bookingId}}
Accept: application/json
Authorization: Bearer {{authToken}}
```

```js 
// Post-request script to confirm cancellation
if (response.status === 204) { // 204 No Content is common for successful DELETE
    console.log(`Booking ${bookingId} cancelled successfully.`);
    bookingId = ""; // Clear the booking ID as it's no longer valid
} else {
    throw new Error(`Booking cancellation failed: Status ${response.status}, Body: ${response.body}`);
}
```
