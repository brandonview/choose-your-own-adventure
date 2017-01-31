#Choose Your Own Adventure

##Overview
`// TODO: Write a brief summary of how to run the game`
`// TODO: Determine an easily parsable format for StoryPoints and ActionOptions to be stored in text files`

##API

####**StoryPoint**
A class representing a point in the story of the game.

#####Constructor
* StoryPoint(Integer storyPointId) Looks up and creates the StoryPoint from the provided ID

#####Fields
* `private const Integer storyPointId` A unique numeric identifier for this StoryPoint. If storage is implemented in a db, this would correspond to the primary key. For a more primitive implementation, the data for this StoryPoint could be saved in a text file named with this ID.
* `private const String storyText` The story text displayed by the narrator for this story point
* `private const List<Integer> actionOptionIds` A list of actionOptionIds for the ActionOptions that are available at this point in the story
* `private const Boolean isFinal` Indicates whether or not this StoryPoint is the end of a story

#####Methods
* `public String getStoryText()` Getter for storyText
* `public List<Integer> getActionOptionIds()` Getter for actionOptionIds
* `public Boolean getIsFinal()` Getter for isFinal

####**ActionOption**

#####Constructor
* `ActionOption(Integer actionOptionId)` Looks up and creates the ActionOption from the provided ID

#####Fields
* `private const Integer actionOptionId` A unique numeric identifier for this ActionOption. If storage is implemented in a db, this would correspond to the primary key. For a more primitive implementation, the data for this ActionOption could be saved in a text file named with this ID. Has a getter.
* `private const String description` Text describing what action the user would take by selecting this `ActionOption`. Has a getter.
* `private const Integer nextStoryPointId` The ID of the next `StoryPoint` that selecting this `ActionOption` would lead to. Has a getter.

####**GameRunner**

#####Fields
* `private const Map<Integer, ActionOption> cachedActionOptions` A map of ActionOption objects to their unique integer IDs so that we don't have to create the object more than once
* `private const Map<Integer, StoryPoint> cachedStoryPoints` A map of StoryPoint objects to their unique integer IDs so that we don't have to create the object more than once
* `private StoryPoint currentStoryPoint` The current StoryPoint being presented in the game.

#####Methods
* `public void runGame(Integer startStoryPointId)` main method for running the game, given the ID of a StoryPoint to start from.
* `private void displayStoryText()` displays the storyText of the currentStoryPoint. The display method is implemented in the GameRunner rather than the StoryPoint class so that the StoryPoint can remain agnostic to how it's storyText is used and displayed as part of the game.
* `private void displayActionOptions()` Creates an ActionOption objects for each of the actionOptionIds of the currentStoryPoint and prints their description. If the ActionOption with corresponding ID is found in the cache, use that and don't create a new one. If not, call the constructor passing actionOptionId and cache the newly created ActionOption. 
* `private Integer promptForActionOptionSelection()` Prompts the user to select an option and reads from stdin. Returns the storyPointId associated with the selected ActionOption
* `private void setNextStoryPoint(Integer storyPointId)` Sets the currentStoryPoint to the StoryPoint corresponding to the storyPointId argument passed in. If the StoryPoint with corresponding ID is found in the cache, use that and don't create a new one. If not, call the constructor passing storyPointId and cache the newly created StoryPoint.

##Pseudocode
I've included some pseudocode for methods that may be nontrivial.
```
GameRunner {
  public void runGame(Integer startStoryPointId) {
    setNextStoryPoint(startStoryPointId);
    
    // main game loop
    while() {
      displayStoryText();
      
      if (currentActionOption.getIsFinal()) {
        break;
      }
      
      displayActionOptions();
      Integer nextStoryPointId = promptForActionOptionSelection();
      setNextStoryPoint(nextStoryPointId);
    }
  }
}
```
