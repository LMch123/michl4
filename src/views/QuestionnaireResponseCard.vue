<template>
  <div class="response-card"
       :class="{ expanded: showDetails, lastItem: isLastItem, 'has-comments': hasComments }" :style="{ backgroundColor: cardColor }" @click="toggleDetails"
  >
    <!-- Display differently for the last item -->
    <template v-if="isLastItem">
      <!-- Display the Total Score label and value of the total score-->
      <strong>{{ question }}</strong>:{{ answerInteger }}
    </template>
    <!-- Render as a regular card for non-last items -->
    <template v-else>
      <!-- Display question number for non-last items -->
      <p class="question-number">
        {{ questionNumber }}
      </p>
      
      <div v-if="showDetails"> 
        <!-- Display question and answer details when details are expanded -->
        <p class="question">
          {{ question }}
        </p>
        <!-- Display coded answer if available -->
        <p v-if="answerCoding" class="answer">
          {{ answerCoding.code }}. {{ answerCoding.display }}
        </p>
        <!-- Display integer answer if available -->
        <p v-else-if="answerInteger !== null" class="answer">
          {{ answerInteger }}
        </p>
        <!-- Display "No answer provided" if no answer is available -->
        <p v-else class="answer">
          No answer provided
        </p>
        <!-- Display comments  if comments exist -->
        <p v-if="comments.length" class="comments-label">
          {{ $t("Remarks:") }}
        </p> 
        <!-- Display each comment and provide delete button -->
        <div v-for="(commentItem, index) in comments" :key="index" class="comment-container">
          <p class="comment">
            {{ commentItem }}
          </p>
          <button class="btn btn-delete-comment" @click.stop="removeComment(index)">
            &times;
          </button>
        </div>
        <!-- Input field for adding new comment -->
        <textarea v-model="comment" @click.stop />
        <!-- Button to save the new comment -->
        <button class="btn btn-success" @click.stop="saveComment">
          {{ $t("SaveRemarks") }}
        </button>
      </div>
    </template>
  </div>
</template>

  
<script>
export default {
  props: {
    // Response data for the currently selected QuestionnaireResponses
    response: {
    type: Object,
    default: () => ({}),
  },
    // linkIds for all questions in a given QuestionnaireResponse.
    questionNumber: {
      type: Number,
      default: 0, // Set an appropriate default value
    },
    //Adds a comment to a specific question in a QuestionnaireResponse.
    addComment: {
      type: Function,
      default: () => {},
    },
    //Deletes a comment from a specific question in a QuestionnaireResponse.
    deleteComment: {
      type: Function,
      default: () => {},
    },
    // Title of the current questionnaire for determining color scales
    questionnaireTitle: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      // Manage card details visibility and new comment input
      showDetails: false,
      // Holds the text of the new comment input
      comment: "",
    };
  },
  computed: {
    /**
     * @vuese
     * Determines the question text for the current card based on the question linkId.
     * @returns {string} - The text of the question.
     */
    question() {
      const questionObject = this.response.item.find((item) => item.linkId === this.questionLinkId);
      return questionObject ? questionObject.text : "";
    },
    /**
     * @vuese
     * Determines the answer coding for the current card.
     * @returns {Object|null} - The answer coding object or null if not available.
     */
    answerCoding() {
      const answerObject = this.response.item.find((item) => item.linkId === this.questionLinkId);
      return answerObject && answerObject.answer && answerObject.answer.length > 0
        ? answerObject.answer[0].valueCoding
        : null;
    },
    /**
     * @vuese
     * Determines the answer integer for the current card.
     * @returns {number|null} - The answer integer or null if not available.
     */
    answerInteger() {
    const answerObject = this.response.item.find((item) => item.linkId === this.questionLinkId);
    return answerObject && answerObject.answer && answerObject.answer.length > 0
      ? answerObject.answer[0].valueInteger
      : null;
    },
    /**
     * @vuese
     * Retrieves comments for the current card.
     * @returns {Array} - An array of comments for the current card.
     */
    comments() {
      const commentExtensionUrl = "http://example.com/fhir/extensions/comment";
      const commentExtensions = this.response.item
        .filter((item) => item.linkId === this.questionLinkId && item.extension)
        .flatMap((item) => item.extension.filter((ext) => ext.url === commentExtensionUrl))
        .map((ext) => ext.valueString);

      return commentExtensions;
    },
    /**
     * @vuese
     * Checks if there are comments for the current card.
     * @returns {boolean} - True if there are comments, false otherwise.
     */
    hasComments() {
      // Check if there are comments
      return this.comments.length > 0;
    },
    
    /**
     * @vuese
     * Determines the background color of the card based on the questionnaire type and score.
     * @returns {string} - The background color.
     */
    cardColor() {
      if (this.isLastItem) {
        const questionnaireTitle = this.questionnaireTitle;
        //console.log("Total Score 2:",this.answerInteger);
        const totalScore = this.answerInteger;

        // Define color scales based on the title of the questionnaire
        const colorScales = {
        'Oxford Knee Score': { 19: "#d32f2f", 29: '#e69500', 39: '#ffeb3b', 48: "#8bae3b" },
        'Oxford Hip Score': {  19: "#d32f2f", 29: '#e69500', 39: '#ffeb3b', 48: "#8bae3b" },
        'Neck Disability Index': { 4: '#8bae3b', 14: '#b9e87d', 24: '#ffeb3b', 34: '#e69500', 50: '#d32f2f' },
        'Oswestry Disability Index': { 4: "#8bae3b", 14: "#b9e87d", 24: "#ffeb3b", 34: "#e69500", 50: "#d32f2f" },
      };

        // Determine the color based on the defined scale
        let color = 'white';
        if (colorScales[questionnaireTitle]) {
          for (const [threshold, scaleColor] of Object.entries(colorScales[questionnaireTitle])) {
            if (totalScore <= parseInt(threshold)) {
              color = scaleColor;
              break;
            }
          }
        }

        return color;
      } else {
        // For non-last items, use the existing color scale logic
        const answerObject = this.response.item.find((item) => item.linkId === this.questionLinkId);

        if (!answerObject || !answerObject.answer || answerObject.answer.length === 0) {
          return 'white';
        }

        const answer = answerObject.answer[0];

        if (answer.valueCoding) {
          const answerCode = parseInt(answer.valueCoding.code);
          const questionnaireTitle = this.questionnaireTitle;
          const severityScale = this.getSeverityScale(questionnaireTitle);

          if (!severityScale) {
            console.error(`Severity scale not defined for questionnaire: ${questionnaireTitle}`);
            return 'white';
          }

          const index = questionnaireTitle === 'EQ-5D-5L Questionnaire' ? answerCode - 1 : answerCode;
          return severityScale[index] || 'white';
        } else if (answer.valueInteger !== undefined) {
          const answerValue = answer.valueInteger;
          return this.getColorScale(answerValue);
        }

        return 'white';
      }
    },
    /**
     * @vuese
     * Determines the linkId of the current card.
     * @returns {string} - The linkId of the current card.
     */
    questionLinkId() {
      return `${this.questionNumber}`;
    },
    /**
     * @vuese
     * Formats the authored date for display.
     * @returns {string} - The formatted authored date.
     */
    authoredDate() {
      return this.response.authored ? new Date(this.response.authored).toLocaleDateString() : "";
    },
    /**
     * @vuese
     * Checks if the current card is the last item.
     * @returns {boolean} - True if the current card is the last item, false otherwise.
     */
    isLastItem() {
      // Check if the current card corresponds to the last item
      const lastItemLinkId = this.response.item[this.response.item.length - 1].linkId;
      return this.questionLinkId === lastItemLinkId;
    },
    
  },

  methods: {
    /**
     * @vuese
     * Retrieves the severity scale based on the questionnaire title.
     * @arg {string} title - The title of the questionnaire.
     * @returns {Array} - An array representing the severity scale.
     */
    getSeverityScale(title) {
      const severityScales = {
        'Neck Disability Index': ["#8bae3b", "#b9e87d", "#ffeb3b", "#e69500", "#d32f2f", "#a36bb7"],
        'Oswestry Disability Index': ["#8bae3b", "#b9e87d", "#ffeb3b", "#e69500", "#d32f2f", "#a36bb7"],
        'Oxford Knee Score': ["#d32f2f", "#e69500", "#ffeb3b", "#b9e87d", "#8bae3b"],
        'Oxford Hip Score': ["#d32f2f", "#e69500", "#ffeb3b", "#b9e87d", "#8bae3b"],
        'EQ-5D-5L Questionnaire': ["#8bae3b", "#b9e87d", "#ffeb3b", "#e69500", "#d32f2f"]
      };
      return severityScales[title];
    },
    /**
     * @vuese
     * Retrieves the color scale based on the value (last card EQ-5D-5L).
     * @param{number} value - The value.
     * @returns {string} - The color.
     */
    getColorScale(value) {
      const colorRanges = [
        { range: [0, 20], color: "#d32f2f" },
        { range: [21, 40], color: "#e69500" },
        { range: [41, 60], color: "#ffeb3b" },
        { range: [61, 80], color: "#b9e87d" },
        { range: [81, 100], color: "#8bae3b" },
      ];

      for (const { range, color } of colorRanges) {
        if (value >= range[0] && value <= range[1]) {
          return color;
        }
      }

      return "white";
    },
    /**
     * @vuese
     * Toggles the display of the card details.
     */
    toggleDetails() {
      this.showDetails = !this.showDetails;
    },
    /**
     * @vuese
     * Saves a new comment for the card and notifies the parent component.
     */
    saveComment() {
      if (this.addComment) {
        this.comments.push(this.comment);
        this.addComment(this.response.id, this.questionNumber, this.comment);
      }
      this.comment = "";

      this.$el.classList.add('has-comments');
    },
    /**
     * @vuese
     * Removes a comment from the card and notifies the parent component.
     * @param{number} index - The index of the comment to be remove.
     */
    removeComment(index) {
      if (this.deleteComment) {
    // Remove the comment from the array
        this.comments.splice(index, 1);
    // Notify the parent component to delete the comment in the QuestionnaireResponseCard
        this.deleteComment(this.response.id, this.questionNumber, index);
    // Force a reactivity update
        this.$forceUpdate();
        if (this.comments.length === 0) {
        this.$el.classList.remove('has-comments');
      }
      }
    },

  },
};
</script>

<style scoped>

.response-card {
  width: 80px;
  height: 80px;
  margin: 5px;
  padding: 5px;
  cursor: pointer;
  border-radius: 8px;
  display: inline-block;
  text-align: center;
  flex: 1;
  max-width: 400px;
  transition: height 0.3s, width 0.3s;
}

.response-card.has-comments {
  border: 2px solid black; 
}
.response-card.lastItem {
  font-weight: bold;
  background-color: lightblue;
  height: 40px; 
  width: 170px; 
  transition: none; 
  text-align: left;
}

.response-card.lastItem:hover {
  transition: none; 
}

.question-number {
  font-weight: bold;
  margin-bottom: 5px;
}

.authored-date {
  font-size: 10px;
  color: gray;
}

.question {
  font-weight: bold;
  font-size: small;
}

.answer {
  margin-top: 5px;
}

.comments-label {
  font-weight: bold;
  margin-top: 5px;
}

.comment-container {
  margin-top: 5px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.comment {
  margin-top: 5px;
}

.btn-delete-comment {
  background-color: #d9534f;
}

.response-card:not(.lastItem):hover {
  height: auto;
  width: auto;
  transition: width 0.3s, height 0.3s;
}

.expanded {
  height: auto;
  width: auto;
  transition: none; 
}
</style>