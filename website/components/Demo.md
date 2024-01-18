# Demo

## Methods

<!-- @vuese:Demo:methods:start -->
|Method|Description|Parameters|
|---|---|---|
|getQuestionnaireResponses|Fetches QuestionnaireResponse data from MIDATA server and organizes it by subject.|-|
|generateLegend|Generates a legend for color-coded severity scales based on the questionnaire title.|{string} questionnaireTitle - The title of the questionnaire.|
|getColorDescription|Retrieves the color description based on the color code.|{string} color - The color code.|
|shouldDisplayButton|Checks if the "View Graph" button should be displayed for a given questionnaire.|{string} questionnaireUrl - The URL of the questionnaire.|
|isAnyNestedSectionExpanded|Checks if at least one nested section is expanded for a given questionnaire.|{string} questionnaireUrl - The URL of the questionnaire.|
|shouldDisplayGraphForQuestionnaire|Checks if the graph should be displayed for a given questionnaire.|{string} questionnaireUrl - The URL of the questionnaire.|
|viewGraph|Displays the graph for a given questionnaire.|{string} questionnaireUrl - The URL of the questionnaire.|
|clearChartData|Clears the chart data when main accordion is retracted.|-|
|getPatient|Fetches patient information from MIDATA server.|-|
|getQuestionnaires|Fetches questionnaires associated with the QuestionnaireResponses.|-|
|selectQuestionnaire|Selects a questionnaire and fetches the associated QuestionnaireResponses.|{string} questionnaireUrl - The URL of the questionnaire.|
|getQuestionnaireResponsesByQuestionnaire|Fetches QuestionnaireResponses for a given questionnaire.|{string} questionnaireUrl - The URL of the questionnaire.|
|getQuestionNumbers|Retrieves the linkIds for all questions in a given QuestionnaireResponse.|{Object} response - The QuestionnaireResponse object.|
|sortedQuestionnaireResponses|Sorts the QuestionnaireResponses by authored date.|{string} questionnaireUrl - The URL of the questionnaire.|
|onAccordionChange|Handles the change event for the accordion input. (expand/collapse).|{string} questionnaireUrl - The URL of the questionnaire.|
|addComment|Adds a comment to a specific question in a QuestionnaireResponse.|{string} responseId - The ID of the QuestionnaireResponse. {number} questionNumber - The number of the question. {string} comment - The comment to be added. {string} questionnaireUrl - The URL of the questionnaire.|
|deleteComment|Deletes a comment from a specific question in a QuestionnaireResponse.|{string} responseId - The ID of the QuestionnaireResponse. {number} questionNumber - The number of the question. {number} index - The index of the comment to be deleted. {string} questionnaireUrl - The URL of the questionnaire.|
|updateTotalScore|Updates the QuestionnaireResponse with the total score if not already exist based on specific calculations.|{Object} response - The QuestionnaireResponse object.|
|calculateTotalScoreSum|Calculates the total score based on the sum of "code" in valueCoding answers.|{Object} response - The QuestionnaireResponse object.|
|calculateTotalScoreConcatenate|Calculates the total score based on the concatenation of the first five "code" in answer valueCoding.|{Object} response - The QuestionnaireResponse object.|
|isSumScoreQuestionnaire|Checks if the title corresponds to a questionnaire using the sum formula.|{string} title - The title of the questionnaire.|
|isConcatenateScoreQuestionnaire|Checks if the title corresponds to a questionnaire using the concatenate formula (EQ-5D-5L).|{string} title - The title of the questionnaire.|
|getQuestionnaireTitle|Retrieves the title of the questionnaire associated with a given QuestionnaireResponse.|{Object} response - The QuestionnaireResponse object.|
|isEQ5D5LQuestionnaire|Checks if the questionnaire title contains "EQ-5D-5L".|{string} title - The title of the questionnaire.|

<!-- @vuese:Demo:methods:end -->


