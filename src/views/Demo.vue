<template> 
  <div>
    <!-- Page Title and Description -->
    <h1>{{ $t("title") }}</h1>
    <p>{{ $t("description") }}</p>
    <!-- Display Patient Information if available -->
    <div v-if="patient && patient.name && patient.name.length" class="alert alert-info mt-3">
      <b>{{ patient.name[0].given[0] }} {{ patient.name[0].family }} </b>
    </div>
    <!-- Loop through QuestionnaireResponses organized by subject to display the name of the patient (Health professionnal view)-->
    <div v-for="(responses, subjectReference) in questionnaireResponsesBySubject" :key="subjectReference">
      <!-- Display subject's (patient) name -->
      <h2>{{ responses[0].subject.display }}</h2> 
      
      <!-- Main Accordion menu for Questionnaires -->
      <div class="accordion"> 
        <!-- Section for each Questionnaire -->
        <div v-for="questionnaire in questionnaires" :key="questionnaire.id" class="accordion-section">
          <!-- Checkbox for expanding/collapsing the Questionnaire section -->
          <input
            :id="`section-${questionnaire.id}`"
            v-model="selectedQuestionnaires" 
            type="checkbox"
            class="accordion-input"
            :value="questionnaire.url" 
            @change="onAccordionChange(questionnaire.url)"
          >
          <!-- Label for Questionnaire section -->
          <label :for="`section-${questionnaire.id}`" class="accordion-label" @click="selectQuestionnaire(questionnaire.url)">
            <strong>{{ questionnaire.title }}</strong>
          <!-- Button to view the graph (disabled if nested sections are not expanded) -->
          </label>
          <!-- Display content only if the current questionnaire is selected -->
          <div v-show="selectedQuestionnaires.includes(questionnaire.url)" class="accordion-content">
            <!-- Nested Accordion menu for each authoredDate -->
            <!-- Button to view the graph (disabled if nested sections (authoredDate) are not expanded) -->
            <!-- Check if the questionnaire title contains "EQ-5D-5L" and don't display View Graph button if so (Concatenate total score)-->
            <button v-if="shouldDisplayButton(questionnaire.url) && !isEQ5D5LQuestionnaire(questionnaire.title)"
                    :disabled="!isAnyNestedSectionExpanded(questionnaire.url)" 
                    @click="viewGraph(questionnaire.url)"
            >
              {{ $t("ViewGraph") }}
            </button>
            <!-- Legend for the severity scale of the questionnaire -->
            <div v-if="severityScales[questionnaire.title]" class="legend-container">
              <!-- Display colored circles with severity descriptions -->
              <div v-for="legendItem in generateLegend(questionnaire.title)" :key="legendItem.color" class="legend-item">
                <div class="legend-circle" :style="{ backgroundColor: legendItem.color }" />
                <div class="legend-description">
                  {{ legendItem.description }}
                </div>
              </div>
            </div>
            <!-- Display TotalScoreGraph component if conditions are met -->
            <div v-if="shouldDisplayGraphForQuestionnaire(questionnaire.url)" class="graph-container">
              <TotalScoreGraph
                :chart-data="chartData"
                :questionnaire-title="questionnaire.title"
              /> 
            </div>
            <!-- Display QuestionnaireResponse cards if there are responses for the current questionnaire -->
            <div v-if="questionnaireResponses[questionnaire.url] && questionnaireResponses[questionnaire.url].length" class="mt-3 sorted-responses">
              <div v-for="(res, responseIndex) in sortedQuestionnaireResponses(questionnaire.url)" :key="res.id">
                <!-- Display accordion section for each authoredDate -->
                <div class="authored-date-section">
                  <!-- Checkbox for expanding/collapsing the authoredDate section -->
                  <input
                    :id="`response-${responseIndex}-section-${questionnaire.id}`"
                    v-model="responseAccordionSections[questionnaire.url][res.id]"
                    type="checkbox"
                    class="authored-date-input"
                  >
                  <!-- Label for authoredDate section -->
                  <label :for="`response-${responseIndex}-section-${questionnaire.id}`" class="authored-date-label">
                    {{ formattedAuthoredDate(res.authored) }}
                  </label>
                  <!-- Display the content if the authoredDate section is expanded -->
                  <div v-show="responseAccordionSections[questionnaire.url][res.id]" class="accordion-content">
                    <!-- Display QuestionnaireResponse cards for each question -->
                    <div class="response-line">
                      <template v-for="(questionNumber, index) in getQuestionNumbers(res)" :key="`r${responseIndex}q${questionNumber}`">
                        <!-- QuestionnaireResponseCard component -->
                        <QuestionnaireResponseCard
                          v-if="index !== getQuestionNumbers(res).length - 1"
                          :response="res"
                          :question-number="questionNumber"
                          :add-comment="addComment"
                          :delete-comment="deleteComment"
                          :questionnaire-title="getQuestionnaireTitle(res)"
                        />
                      </template>
                    </div>
                    <!-- Display the last card separately below the first item -->
                    <QuestionnaireResponseCard
                      :response="res"
                      :question-number="getQuestionNumbers(res)[getQuestionNumbers(res).length - 1]"
                      :add-comment="addComment"
                      :delete-comment="deleteComment"
                      :questionnaire-title="getQuestionnaireTitle(res)"
                    />
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import midataServer from "vue-midata/midata.js";
import midataPortal from "vue-midata/midataPortal.js";
import QuestionnaireResponseCard from "views/QuestionnaireResponseCard.vue";
import TotalScoreGraph from "views/TotalScoreGraph.vue";

export default {
  // Define components used in this Vue component
  components: {
    // QuestionnaireResponseCard component for displaying individual questionnaire responses
    QuestionnaireResponseCard,
    // TotalScoreGraph component for displaying the total score graph
    TotalScoreGraph,
  },
  data() {
    return {
      patient: {}, // Patient information retrieved from MIDATA server
      questionnaireResponses: {}, // QuestionnaireResponses organized by the associated questionnaire URL
      accordionSections: {}, // Accordion sections for each response indicating expanded/collapsed state
      responseAccordionSections: {}, // Accordion sections for nested responses indicating expanded/collapsed state
      selectedQuestionnaires: [], // User-selected Questionnaire URLs
      questionnaires: [], // Questionnaire data fetched from MIDATA server
      selectedResponsesForGraph: [], // // Selected QuestionnaireResponses for displaying in the graph
      shouldDisplayGraph: false, // Flag indicating whether the graph should be displayed
      chartData: {}, // Data for the graph (chart) displayed using TotalScoreGraph component
      response: {}, // Response data for the currently selected QuestionnaireResponses
      // Severity scales for different questionnaires (for the legend)
      severityScales: {
      'Neck Disability Index': ["#8bae3b", "#b9e87d", "#ffeb3b", "#e69500", "#d32f2f", "#a36bb7"],
      'Oswestry Disability Index': ["#8bae3b", "#b9e87d", "#ffeb3b", "#e69500", "#d32f2f", "#a36bb7"],
      'Oxford Knee Score': ["#8bae3b", "#b9e87d", "#ffeb3b", "#e69500", "#d32f2f"],
      'Oxford Hip Score': ["#8bae3b", "#b9e87d", "#ffeb3b", "#e69500", "#d32f2f"],
      'EQ-5D-5L Questionnaire': ["#8bae3b", "#b9e87d", "#ffeb3b", "#e69500", "#d32f2f"],
    },
      //isLastItemDisplayed: false,
      questionnaireResponsesBySubject: {}, // Organized QuestionnaireResponse data grouped by subject reference
      
    };
  },

  mounted() {
    // Call the method to get patient information
    this.getPatient();
    this.getQuestionnaireResponses()
    .then(() => {
      // Call getQuestionnaires() after getQuestionnaireResponses() is completed
      this.getQuestionnaires();
    })
    .catch(error => {
      console.error("Error fetching questionnaire responses:", error);
    });
  },

  methods: {
    /**
     * @vuese
     * Fetches QuestionnaireResponse data from MIDATA server and organizes it by subject.
     * @returns {Promise<void>} - Resolves when the QuestionnaireResponse data is fetched and organized.
     */
    async getQuestionnaireResponses() {
    // Fetch all questionnaire responses
    const responses = await midataServer.fhirSearch("QuestionnaireResponse", {}, true);

    // Organize responses by subject
    const responsesBySubject = {};
    responses.forEach(response => {
      const subjectReference = response.subject && response.subject.reference;
      if (subjectReference) {
        responsesBySubject[subjectReference] = responsesBySubject[subjectReference] || [];
        responsesBySubject[subjectReference].push(response);
      }
    });
    // Save organized responses
    this.questionnaireResponsesBySubject = responsesBySubject;
  },
   /**
     * @vuese
     * Generates a legend for color-coded severity scales based on the questionnaire title.
     * @arg {string} questionnaireTitle - The title of the questionnaire.
     * @returns {Array} - An array containing color and description objects for the legend.
     */
    generateLegend(questionnaireTitle) {
    const scale = this.severityScales[questionnaireTitle] || [];
    const legend = scale.map(color => ({
      color,
      description: this.getColorDescription(color),
    }));
    return legend;
  },
    /**
     * @vuese
     * Retrieves the color description based on the color code.
     * @arg {string} color - The color code.
     * @returns {string} - The description corresponding to the color.
     */
    getColorDescription(color) {
    // Define descriptions for each color with i18n keys
    const descriptions = {
      "#d32f2f": this.$t("VerySevereProblems"),
      "#e69500": this.$t("SevereProblems"),
      "#ffeb3b": this.$t("ModerateProblems"),
      "#b9e87d": this.$t("MildProblems"),
      "#8bae3b": this.$t("NoProblem"),
      "#a36bb7": this.$t("ExtremelySevereProblems"),
    };

    return descriptions[color] || "";
  },
    /**
     * @vuese
     * Checks if the "View Graph" button should be displayed for a given questionnaire.
     * @arg {string} questionnaireUrl - The URL of the questionnaire.
     * @returns {boolean} - True if the button should be displayed, false otherwise.
     */
    shouldDisplayButton(questionnaireUrl) {
      // Check if the button should be displayed for the given questionnaire
      return this.selectedQuestionnaires.includes(questionnaireUrl);
    },
    /**
     * @vuese
     * Checks if at least one nested section is expanded for a given questionnaire.
     * @arg {string} questionnaireUrl - The URL of the questionnaire.
     * @returns {boolean} - True if at least one nested section is expanded, false otherwise.
     */
    isAnyNestedSectionExpanded(questionnaireUrl) {
      // Check if at least one nested section is expanded
      return Object.values(this.responseAccordionSections[questionnaireUrl] || {}).some((expanded) => expanded);
    },
    /**
     * @vuese
     * Checks if the graph should be displayed for a given questionnaire.
     * @arg {string} questionnaireUrl - The URL of the questionnaire.
     * @returns {boolean} - True if the graph should be displayed, false otherwise.
     */
    shouldDisplayGraphForQuestionnaire(questionnaireUrl) {
      return this.shouldDisplayGraph && this.selectedQuestionnaires.includes(questionnaireUrl);
    },
    /**
     * @vuese
     * Displays the graph for a given questionnaire.
     * @arg {string} questionnaireUrl - The URL of the questionnaire.
     */
    viewGraph(questionnaireUrl) {
      const responses = this.questionnaireResponses[questionnaireUrl] || [];
      this.selectedResponsesForGraph = responses.filter(response => this.responseAccordionSections[questionnaireUrl][response.id]);
      console.log('Selected Responses for Graph:', this.selectedResponsesForGraph);

      // Extract data for the graph
      const graphAuthoredDates = this.selectedResponsesForGraph.map(response => this.formattedAuthoredDate(response.authored));
      const graphTotalScores = this.selectedResponsesForGraph.map(response => {
      const lastItem = response.item[response.item.length - 1];
      return lastItem && lastItem.answer && lastItem.answer[0] && lastItem.answer[0].valueInteger || 0;
      });

      // Set data properties to display the graph
      this.shouldDisplayGraph = true;

          // Set the chart data directly in TotalScoreGraph
          this.chartData = {
            labels: graphAuthoredDates,
            datasets: [
              {
                label: 'Total Score',
                //backgroundColor: 'blue',
                data: graphTotalScores,
              },
            ],
          };
          console.log("Chart Data:",this.chartData);
    },
    /**
     * @vuese
     * Clears the chart data when main accordion is retracted.
     */
    clearChartData() {
      this.shouldDisplayGraph = false;
      this.chartData = {};
    },
    /**
     * @vuese
     * Fetches patient information from MIDATA server.
     */
    getPatient() {
      midataServer.fhirSearch("Patient", { _id: midataPortal.owner }, true)
        .then((result) => (this.$data.patient = result[0]));
    },
    /**
     * @vuese
     * Fetches questionnaires associated with the QuestionnaireResponses.
     */
    getQuestionnaires() {
      // Get all questionnaire URLs from the questionnaire responses
      const questionnaireUrls = Object.values(this.questionnaireResponsesBySubject)
        .flatMap(responses => responses.map(response => response.questionnaire))
        .filter(url => url);
        console.log("questionnaire Responses BySubject:" ,this.questionnaireResponsesBySubject);

      // Remove duplicate questionnaire URLs using Set
      const uniqueQuestionnaireUrls = [...new Set(questionnaireUrls)];

      // Fetch the questionnaires using the unique URLs
      Promise.all(uniqueQuestionnaireUrls.map(url => midataServer.fhirSearch("Questionnaire", { url }, true)))
        .then(questionnairesArray => {
          // Flatten the array of arrays into a single array of questionnaires
          const questionnaires = [].concat(...questionnairesArray);
          this.$data.questionnaires = questionnaires;
          // Log the questionnaires array to the console
          console.log('Questionnaires Array:', this.questionnaires);
        })
        .catch(error => {
          console.error("Error fetching questionnaires:", error);
        });
    },
    /**
     * @vuese
     * Selects a questionnaire and fetches the associated QuestionnaireResponses.
     * @arg {string} questionnaireUrl - The URL of the questionnaire.
     */
    selectQuestionnaire(questionnaireUrl) {
      // Initialize responseAccordionSections for the selected questionnaire
      if (!this.responseAccordionSections[questionnaireUrl]) {
        this.responseAccordionSections = {
          ...this.responseAccordionSections,
          [questionnaireUrl]: {},
        };
      }
      // Fetch QuestionnaireResponses for the selected questionnaire
      this.getQuestionnaireResponsesByQuestionnaire(questionnaireUrl);
    },
    /**
     * @vuese
     * Fetches QuestionnaireResponses for a given questionnaire.
     * @arg {string} questionnaireUrl - The URL of the questionnaire.
     */
    getQuestionnaireResponsesByQuestionnaire(questionnaireUrl) {
      midataServer.fhirSearch("QuestionnaireResponse", { questionnaire: questionnaireUrl, }, true)
        .then((responses) => {
          // Update the questionnaireResponses object with the fetched responses
          this.questionnaireResponses[questionnaireUrl] = responses.reverse();

          // Iterate through each response and update with total score if not exist already
          this.questionnaireResponses[questionnaireUrl].forEach(response => {
            this.updateTotalScore(response);
          });
        });
        console.log("Questionnaire Response:", this.questionnaireResponses);
    },
    /**
     * @vuese
     * Retrieves the linkIds for all questions in a given QuestionnaireResponse.
     * @arg {Object} response - The QuestionnaireResponse object.
     * @returns {Array} - An array containing the linkIds for all questions.
     */
    getQuestionNumbers(response) {
      return response.item.map((item) => parseInt(item.linkId));
    },
    /**
     * @vuese
     * Sorts the QuestionnaireResponses by authored date.
     * @arg {string} questionnaireUrl - The URL of the questionnaire.
     * @returns {Array} - An array containing the sorted QuestionnaireResponses.
     */
    sortedQuestionnaireResponses(questionnaireUrl) {
      const allResponses = this.questionnaireResponses[questionnaireUrl] || [];

      // Filter responses for the selected questionnaire
      const selectedResponses = allResponses.filter(response => this.responseAccordionSections[questionnaireUrl][response.id]);
      //console.log("Selected Responses:",selectedResponses);
      //console.log("Response Accordion Sections:", this.responseAccordionSections );
      // Sort the selected responses by authored date
      const sortedSelectedResponses = selectedResponses.slice().sort((a, b) => {
        const dateA = new Date(a.authored);
        const dateB = new Date(b.authored);
        return dateA - dateB;
      }); 

      // Concatenate responses in chronological order
      const sortedResponses = sortedSelectedResponses.concat(allResponses.filter(response => !selectedResponses.includes(response)));
      //console.log("Sorted Responses:",sortedResponses);
      return sortedResponses;

    },
    /**
     * @vuese
     * Handles the change event for the accordion input. (expand/collapse).
     * @arg {string} questionnaireUrl - The URL of the questionnaire.
     */
    onAccordionChange(questionnaireUrl) {
    // Check if the accordion input is unchecked
    if (!this.selectedQuestionnaires.includes(questionnaireUrl)) {
      // Clear the chart data when the accordion section is closed
      this.clearChartData();
    }
    console.log("Selected Questionnaires:",this.selectedQuestionnaires);
  },

    //AuthoredDate for the nested accordion labels
    formattedAuthoredDate(authoredDate) {
      //console.log("Authored Date:",authoredDate);
      return authoredDate ? new Date(authoredDate).toLocaleDateString() : "";
    },
    /**
     * @vuese
     * Adds a comment to a specific question in a QuestionnaireResponse.
     * @arg {string} responseId - The ID of the QuestionnaireResponse.
     * @arg {number} questionNumber - The number of the question.
     * @arg {string} comment - The comment to be added.
     * @arg {string} questionnaireUrl - The URL of the questionnaire.
     */
    addComment(responseId, questionNumber, comment, questionnaireUrl) {
      // Fetch the latest version of the QuestionnaireResponse
      midataServer.fhirSearch("QuestionnaireResponse", { _id: responseId, _sort: "-_lastUpdated" ,patient: midataPortal.owner}, true)
        .then((responses) => {
          const latestResponse = responses[0];

          // Find the specific item/question in the response
          const questionItem = latestResponse.item.find((item) => parseInt(item.linkId) === questionNumber);
          if (questionItem) {
            // Add the comment to the extension or answer as needed
            questionItem.extension = questionItem.extension || [];
            questionItem.extension.push({
              url: "http://example.com/fhir/extensions/comment",
              valueString: comment,
            });

            // Save the updated QuestionnaireResponse
            midataServer.fhirUpdate(latestResponse)
              .then(() => {
                console.log("Comment saved successfully");
                // Fetch the updated QuestionnaireResponses for the selected questionnaire
                this.getQuestionnaireResponsesByQuestionnaire(questionnaireUrl);
              })
              .catch((error) => {
                console.error("Error saving comment:", error);
              });
          }
        })
        .catch((error) => {
          console.error("Error fetching latest version:", error);
        });
    },
    /**
     * @vuese
     * Deletes a comment from a specific question in a QuestionnaireResponse.
     * @arg {string} responseId - The ID of the QuestionnaireResponse.
     * @arg {number} questionNumber - The number of the question.
     * @arg {number} index - The index of the comment to be deleted.
     * @arg {string} questionnaireUrl - The URL of the questionnaire.
     */
    deleteComment(responseId, questionNumber, index, questionnaireUrl) {
      // Fetch the latest version of the QuestionnaireResponse
      midataServer.fhirSearch("QuestionnaireResponse", { _id: responseId, _sort: "-_lastUpdated" ,patient: midataPortal.owner}, true)
        .then((responses) => {
          const latestResponse = responses[0];

          // Find the specific item/question in the response
          const questionItem = latestResponse.item.find((item) => parseInt(item.linkId) === questionNumber);
          if (questionItem && questionItem.extension && questionItem.extension.length > index) {
            // Remove the comment from the extension
            questionItem.extension.splice(index, 1);

            // Save the updated QuestionnaireResponse
            midataServer.fhirUpdate(latestResponse)
              .then(() => {
                console.log("Comment deleted successfully");
                // Fetch the updated QuestionnaireResponses for the selected questionnaire
                this.getQuestionnaireResponsesByQuestionnaire(questionnaireUrl);
              })
              .catch((error) => {
                console.error("Error deleting comment:", error);
              });
          }
        })
        .catch((error) => {
          console.error("Error fetching latest version:", error);
        });
    },
    /**
     * @vuese
     * Updates the QuestionnaireResponse with the total score if not already exist based on specific calculations.
     * @arg {Object} response - The QuestionnaireResponse object.
     */
    updateTotalScore(response) {
      console.log("Questionnaire URL:", response.questionnaire);

      // Check if the total score is already present
      const hasTotalScore = response.item.some(item => item.text === 'Total Score');

    if (!hasTotalScore) {
      // Determine the title of the questionnaire
      const questionnaireTitle = this.getQuestionnaireTitle(response);

      // Calculate the total score based on the formula
      let totalScore;
      if (this.isSumScoreQuestionnaire(questionnaireTitle)) {
        totalScore = this.calculateTotalScoreSum(response);
      } else if (this.isConcatenateScoreQuestionnaire(questionnaireTitle)) {
        totalScore = this.calculateTotalScoreConcatenate(response);
      } else {
        console.error(`Unsupported questionnaire title: ${questionnaireTitle}`);
        return;
      }

      // Find the maximum linkId in the existing items and increment it by 1
      const maxLinkId = response.item.reduce((max, item) => Math.max(max, parseInt(item.linkId) || 0), 0);
      const totalScoreLinkId = (maxLinkId + 1).toString();

      // Create a new item for the total score with the new calculated Total Score linkId
      const totalScoreItem = {
        linkId: totalScoreLinkId,
        text: 'Total Score',
        answer: [{ valueInteger: totalScore }],
      };

      // Add the total score item to the QuestionnaireResponse
      response.item.push(totalScoreItem);

      // Update the QuestionnaireResponse on the server
      midataServer.fhirUpdate(response)
        .then(() => {
          console.log("QuestionnaireResponse updated successfully with total score");
          // Fetch the updated QuestionnaireResponses for the selected questionnaire
          this.getQuestionnaireResponsesByQuestionnaire(response.questionnaire);
        })
        .catch((error) => {
          console.error("Error updating QuestionnaireResponse:", error);
        });
    }
  },
  /**
   * @vuese
   * Calculates the total score based on the sum of "code" in valueCoding answers.
   * @arg {Object} response - The QuestionnaireResponse object.
   * @returns {number} - The calculated total score.
   */
  calculateTotalScoreSum(response) {
    // Calculate total score based on the sum of "code" in valueCoding answer
    let totalScore = 0;

    response.item.forEach((item) => {
      if (item.answer && item.answer[0] && item.answer[0].valueCoding && item.answer[0].valueCoding.code) {
        totalScore += parseInt(item.answer[0].valueCoding.code) || 0;
      }
    });

    return totalScore;
  },
  /**
   * @vuese
   * Calculates the total score based on the concatenation of the first five "code" in answer valueCoding.
   * @arg {Object} response - The QuestionnaireResponse object.
   * @returns {string} - The calculated total score.
   */
  calculateTotalScoreConcatenate(response) {
    // Calculate total score based on the concatenation of the first five "code" in answer valueCoding (EQ-5D-5L Questionnaire)
    let totalScore = '';

    for (let i = 1; i <= 5; i++) {
      const item = response.item.find((item) => parseInt(item.linkId) === i);

      if (item && item.answer && item.answer[0] && item.answer[0].valueCoding && item.answer[0].valueCoding.code) {
        totalScore += item.answer[0].valueCoding.code;
      } else {
        // Handle the case where an expected answer is missing
        console.error(`Answer for question ${i} is missing or invalid.`);
      }
    }
    //console.log("Total Score:",totalScore);
    return totalScore;
  },
  /**
   * @vuese
   * Checks if the title corresponds to a questionnaire using the sum formula.
   * @arg {string} title - The title of the questionnaire.
   * @returns {boolean} - True if the title corresponds to a questionnaire using the sum formula, false otherwise.
   */
  isSumScoreQuestionnaire(title) {
    // Check if the title corresponds to a questionnaire using the sum formula
    const sumScoreQuestionnaires = ["Oxford Knee Score", "Oxford Hip Score", "Neck Disability Index", "Oswestry Disability Index"];
    return sumScoreQuestionnaires.includes(title);
  },
  /**
   * @vuese
   * Checks if the title corresponds to a questionnaire using the concatenate formula (EQ-5D-5L).
   * @arg {string} title - The title of the questionnaire.
   * @returns {boolean} - True if the title corresponds to a questionnaire using the concatenate formula, false otherwise.
   */
  isConcatenateScoreQuestionnaire(title) {
    // Check if the title corresponds to a questionnaire using the concatenate formula
    return title === "EQ-5D-5L Questionnaire";
  },
  /**
   * @vuese
   * Retrieves the title of the questionnaire associated with a given QuestionnaireResponse.
   * @arg {Object} response - The QuestionnaireResponse object.
   * @returns {string} - The title of the questionnaire.
   */
  getQuestionnaireTitle(response) {
    const questionnaire = this.questionnaires.find((q) => q.url === response.questionnaire);
    return questionnaire ? questionnaire.title : "";
  },
  /**
   * @vuese
   * Checks if the questionnaire title contains "EQ-5D-5L".
   * @arg{string} title - The title of the questionnaire.
   * @returns {boolean} - True if the title contains "EQ-5D-5L", false otherwise.
   */
  isEQ5D5LQuestionnaire(title) {
    // Check if the title contains "EQ-5D-5L"
    return title.toLowerCase().includes("eq-5d-5l");
  },

  },
};
</script>

<style scoped>
  /* Existing styles for the accordion menu */
  h1 {
    color: red;
  }
  
  .accordion {
    width: 100%;
  }

  .accordion-section {
    border: 1px solid #ddd;
    margin-bottom: 10px;
    border-radius: 4px;
    overflow: hidden;
  }

  .accordion-input {
    display: none;
  }

  .accordion-label {
    display: block;
    padding: 10px;
    cursor: pointer;
    background-color: #f5f5f5;
  }

  .accordion-content {
    max-height: 0;
    overflow: auto;
    transition: max-height 0.3s ease-out;
  }

  .accordion-input:checked + .accordion-label + .accordion-content {
    max-height: 1000px; 
  }

  .response-line {
    display: flex;
    flex-wrap: nowrap;
    margin-bottom: none;
  }

  .authored-date-section {
    border: 1px solid #ddd;
    margin-bottom: 10px;
    border-radius: 4px;
    overflow: hidden;
  }

  .authored-date-label {
    display: block;
    padding: 10px;
    cursor: pointer;
    background-color: #f5f5f5;
  }

  .authored-date-label:hover {
    background-color: #e5e5e5;
  }
  .authored-date-input{
    display: none;
  }
  .authored-date-input:checked + .authored-date-label + .accordion-content {
    max-height: 1000px; 
  }

  .sorted-responses {
    display: flex;
    flex-direction: column;
    margin-top: 10px;
  }

  .graph-container {
    max-width: 80%; /* Adjust the maximum width as needed */
    margin: 0 auto; /* Center the graph container */
    padding: 10px; /* Add padding as needed */
  }

  .legend-container {
  display: flex; /* Use flexbox to create a row layout */
  align-items: center; /* Center items vertically */
  margin-top: 10px;
}

.legend-item {
  display: flex;
  align-items: center;
  margin-right: 15px; /* Adjust the margin between legend items as needed */
}

.legend-circle {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  margin-right: 5px;
}

.legend-description {
  font-size: small;
}

</style>