# QuestionnaireResponseCard

## Props

<!-- @vuese:QuestionnaireResponseCard:props:start -->
|Name|Description|Type|Required|Default|
|---|---|---|---|---|
|response|Response data for the currently selected QuestionnaireResponses|`Object`|`false`|{}|
|questionNumber|linkIds for all questions in a given QuestionnaireResponse.|`Number`|`false`|0|
|addComment|Adds a comment to a specific question in a QuestionnaireResponse.|`Function`|`false`|() => {}|
|deleteComment|Deletes a comment from a specific question in a QuestionnaireResponse.|`Function`|`false`|() => {}|
|questionnaireTitle|Title of the current questionnaire for determining color scales|`String`|`false`|-|

<!-- @vuese:QuestionnaireResponseCard:props:end -->


## Methods

<!-- @vuese:QuestionnaireResponseCard:methods:start -->
|Method|Description|Parameters|
|---|---|---|
|getSeverityScale|Retrieves the severity scale based on the questionnaire title.|{string} title - The title of the questionnaire.|
|getColorScale|Retrieves the color scale based on the value (last card EQ-5D-5L).|-|
|toggleDetails|Toggles the display of the card details.|-|
|saveComment|Saves a new comment for the card and notifies the parent component.|-|
|removeComment|Removes a comment from the card and notifies the parent component.|-|

<!-- @vuese:QuestionnaireResponseCard:methods:end -->


## Computed

<!-- @vuese:QuestionnaireResponseCard:computed:start -->
|Computed|Type|Description|From Store|
|---|---|---|---|
|question|-|Determines the question text for the current card based on the question linkId.|No|
|answerCoding|-|Determines the answer coding for the current card.|No|
|answerInteger|-|Determines the answer integer for the current card.|No|
|comments|-|Retrieves comments for the current card.|No|
|hasComments|-|Checks if there are comments for the current card.|No|
|cardColor|-|Determines the background color of the card based on the questionnaire type and score.|No|
|questionLinkId|-|Determines the linkId of the current card.|No|
|authoredDate|-|Formats the authored date for display.|No|
|isLastItem|-|Checks if the current card is the last item.|No|

<!-- @vuese:QuestionnaireResponseCard:computed:end -->


