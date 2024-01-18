# PROMsQuest MIDATA Portal Plugin

PROMsQuest is a MIDATA portal plugin developed using Vue.js. It is designed to display Patient Reported Outcome Measures (PROMs) in a user-friendly manner. The plugin supports the following questionnaires: Oxford Knee Score, Oxford Hip Score, Oswestry Disability Index, Neck Disability Index, and EQ-5D-5L.

## Getting Started

1. Clone this repository into a directory on your PC.
2. Run "npm install" in that directory.
4. Run "npm run serve."
5. Open the plugin in your browser at URL https://localhost:9004/promsquest/index.html. Your browser may complain about the SSL certificate; add a security exception. If successful, you should see PROMsQuest in your browser.
6. Find the PROMsQuest tile; click "view." The plugin should now be visible inside the portal. If issues arise, verify step 5 and browser security exceptions.

## Accessing Test Users

To test the application using test user accounts, you can access the MIDATA test instance at the following address: [https://test.midata.coop/#/public/info](https://test.midata.coop/#/public/info).

Test user credentials are available in the public files. Please refer to the relevant files in the repository for details on the test user accounts.

## Application Details

Additional documentation for the PROMsQuest plugin can be found in the "website" folder within the "src" directory. 

### Demo.vue

- **Purpose:** Main component responsible for handling communication with MIDATA and organizing/rendering the user interface.
- **Key Features:**
  - Communicates with the MIDATA API using the [midata-plugin-vue-library](https://github.com/MIDATAcooperative/midata-plugin-library) to retrieve Patient Reported Outcome Measures (PROMs) data.
  - Organizes and structures the user interface for seamless user interaction.
  - Coordinates data presentation through child components like QuestionnaireResponseCard.vue and TotalScoreGraph.vue.
  - Implements dynamic rendering of PROMs data using colored cards based on severity.
  - Facilitates interaction with MIDATA for data updates and synchronization.

### QuestionnaireResponseCard.vue

- **Purpose:** Reusable component for displaying questionnaire response cards.
- **Key Features:**
  - Dynamically renders information about a questionnaire response.
  - Handles the display of questions, answers, and comments.
  - Applies color coding based on questionnaire type and score.
  - Allows users to add, delete, and save comments.

### TotalScoreGraph.vue

- **Purpose:** Component for rendering a total score graph.
- **Key Features:**
  - Utilizes Vue Chart.js library to create a bar chart.
  - Dynamically adjusts Y-axis max value based on questionnaire type.
  - Colors bars based on total scores and predefined color scales.
  - Integrates with MIDATA portal for visualizing total scores.
  