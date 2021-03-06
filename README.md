#Choose Your Own Adventure

##Overview
A choose your own adventure game is an interactive story where the plot and ending are influenced by the reader's choices throughout the game.

There is no code in this repository as it was created as an exercise in design while I was bored at work. Currently this repository serves only as a host to this README.md file outlining a reasonably well thought out (well I thought so anyway) API for how this game might be designed in a Java-like object oriented language.

##API

####**StoryPoint**
A class representing a point in the story of the game.

#####Constructor
* `StoryPoint(Integer storyPointId)` Looks up and creates the StoryPoint from the provided ID

#####Fields
* `private final Integer storyPointId` A unique numeric identifier for this StoryPoint. If storage is implemented in a db, this would correspond to the primary key. For a more primitive implementation, the data for this StoryPoint could be saved in a text file named with this ID.
* `private final String storyText` The story text displayed by the narrator for this story point
* `private final List<Integer> actionOptionIds` A list of actionOptionIds for the ActionOptions that are available at this point in the story
* `private final Boolean isFinal` Indicates whether or not this StoryPoint is the end of a story

#####Methods
* `public String getStoryText()` Getter for storyText
* `public List<Integer> getActionOptionIds()` Getter for actionOptionIds
* `public Boolean getIsFinal()` Getter for isFinal

####**ActionOption**

#####Constructor
* `ActionOption(Integer actionOptionId)` Looks up and creates the ActionOption from the provided ID

#####Fields
* `private final Integer actionOptionId` A unique numeric identifier for this ActionOption. If storage is implemented in a db, this would correspond to the primary key. For a more primitive implementation, the data for this ActionOption could be saved in a text file named with this ID. Has a getter.
* `private final String description` Text describing what action the user would take by selecting this `ActionOption`. Has a getter.
* `private final Integer nextStoryPointId` The ID of the next `StoryPoint` that selecting this `ActionOption` would lead to. Has a getter.

####**GameRunner**

#####Fields
* `private final Map<Integer, ActionOption> cachedActionOptions` A map of ActionOption objects to their unique integer IDs so that we don't have to create the object more than once
* `private final Map<Integer, StoryPoint> cachedStoryPoints` A map of StoryPoint objects to their unique integer IDs so that we don't have to create the object more than once
* `private StoryPoint currentStoryPoint` The current StoryPoint being presented in the game.

#####Methods
* `public void runGame(Integer startStoryPointId)` main method for running the game, given the ID of a StoryPoint to start from.
* `private void displayStoryText()` displays the storyText of the currentStoryPoint. The display method is implemented in the GameRunner rather than the StoryPoint class so that the StoryPoint can remain agnostic to how its storyText is used and displayed as part of the game.
* `private void displayActionOptions()` Creates an ActionOption objects for each of the actionOptionIds of the currentStoryPoint and prints their description. If the ActionOption with corresponding ID is found in the cache, use that and don't create a new one. If not, call the finalructor passing actionOptionId and cache the newly created ActionOption. 
* `private Integer promptForActionOptionSelection()` Prompts the user to select an option and reads from stdin. Returns the storyPointId associated with the selected ActionOption
* `private void setNextStoryPoint(Integer storyPointId)` Sets the currentStoryPoint to the StoryPoint corresponding to the storyPointId argument passed in. If the StoryPoint with corresponding ID is found in the cache, use that and don't create a new one. If not, call the finalructor passing storyPointId and cache the newly created StoryPoint.

####**StoryPointBuilder**

If we don't mind making our codebase a little bigger in the interest of making the code a little more flexible, 
the correct thing to do would be to have a helper class called something like `StoryPointBuilder` full of static methods
including one that returns a StoryPoint given a storyPointId. 

With this implementation, an all args constructor would replace the current StoryPoint constructor that takes a StoryPointID.

This allows the StoryPoint class to be decoupled from the way we choose to store data for the StoryPoints in the game. 
If we decide to use a database or another storage method than json files, then this would be the only class that needs to change.
The GameRunner class would call StoryPointBuilder.buildFromStoryPointId(storyPointId) which could be modified to implement
different lookup systems without affecting clients of the class.

The StoryPointBuilder would manage constant configuration values like filetype extension for StoryPoint data files 
and the directory path to these files as well as a system level file reader.

##Pseudocode
I've included some pseudocode for methods that may be nontrivial.


Here's what the runGame method might look like.

```java
public class GameRunner {
  public void runGame(Integer startStoryPointId) {
    setNextStoryPoint(startStoryPointId);
    
    // main game loop
    while(true) {
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

Now here's an implementation of how you might create a StoryPoint object from an ID that is the filename.
The file with data corresponding to the StoryPoint should be a format that you can find an already implemented parser for like json or xml.

```java
final String FILETYPE_EXTENSION = ".json";
final String STORYPOINTS_FILES_HOME_DIR = "../data/storypoints/";

// not literally the name of these classes, look some up for whatever language/framework you're using
final FileReader fileReader = new FileReader();
final FileParser fileParser = new JsonFileParser();

public class StoryPoint {
  public StoryPoint(Integer storyPointId) {
    String storyPointFileName = storyPointId + FILETYPE_EXTENSION;
    String storyPointFileFullPath = STORYPOINTS_FILES_HOME_DIR + storyPointFileName;
    
    File storyPointFile = fileReader.open(storyPointFileFullPath);

    if (storyPointFile == null) { 
      throw new Error("No story point with ID ${storyPointID} found"); 
    }

    // Using a std lib or third party parser should return a map with keys for 
    // field names or something like that
    Map<String, object> rawStoryPointData = fileParser.parse(storyPointFile);

    // Parse and assign values from the raw data
    this.storyPointId = storyPointId;
    this.storyText = rawStoryPointData["storyText"];
    this.actionOptionIds = new List<Integer>(rawStoryPointData["actionOptionIds"]);
    this.isFinal = rawStoryPointData["isFinal"];
  }
}
```

