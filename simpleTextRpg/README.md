
#simple-text-rpg

##Overview
This is a readme outlining a potential text based rpg that would run in a console. 

## Class Design

###`public interface ItemHolder`
An interface for anything in the game that can hold an item. This could be a GameArea, a Player, or the GameRunner itself

 - `public void addItem(ItemReferenceName itemReferenceName, int count)`
 - `public Item getItems()`
 - `public Item getItemCount(ItemReferenceName)`
 - `public void deleteItem(ItemReferenceName itemReferenceName)`

###`public interface GameObject`
- `public String getDescription()` Every object that appears in this game should have a description.

###`public class ItemReference implements GameObject`
The class in the game to represent an item. Items in the game are never instantiated, but instead are represented as a number of items and a link to the original reference. Items are referenced by an `ItemReferenceName`

- `public enum ItemReferenceName`
- `public ItemReferenceName getItemReferenceName()`
- `public Boolean isTransferable()` Indicates if this item can be transferred from one ItemHolder to another

###`public class GameArea implements GameObject, ItemHolder`
- Implements Necessary Override methods

###`public class Player implements ItemHolder`
- Implements necessary override methods

###`public abstract final class GameObjectReader`
- `public static Map<String, String> readGameAreaMapConfig(String filename)` 
- `public static GameArea readGameAreaConfig(String filename)`
- `public static GameLocation readGameLocationConfig(String filename)`

###`public class PlayerAction`
- `public enum PlayerActionCodes`
- `public object[] getArgs`
- `public PlayerActionCode getPlayerActionCode()`

###`public class GameRunner implements ItemHolder`
- `public void setPlayer(Player player)`
- `public void setGameAreaMap(Map<GameLocation, GameArea> gameAreaMap)`
- `public void runGameLoop()`

The following is a list of private members that are used in the game loop.

- `private void displayAvailablePlayerActions()` This will be the method that requires some thought about how you'll figure out what options are available
- `private PlayerAction getNextPlayerAction()`
- `private void executePlayer(PlayerActionCode nextAction)`

The following is a list of private methods this class might call after the player makes a choice in the prompt

- `private void describeCurrentGameArea()`
- `private void goToNextGameArea(GameLocation nextLocation)`
- `private void transferItems(ItemHolder sender, ItemHolder recipient, ItemReference item, Integer count)`

##Pseudocode
Now here's some code for all you folks still confused about how this might work.

Main class implementation.
```java
// The main class that creates a game runner then runs the main program loop
int main() {
  // Initialization
  Map<String, String> gameAreaMapConfig = GameObjectReader.readGameAreaMapConfig("mapConfig.json");
  Map<GameLocation, GameArea> gameAreaMap = new Map<GameLocation, GameArea>();
  for (String gameLocationConfig : gameAreaMapConfig.getKeys()) {
    gameAreaMap.add( 
            GameObjectReader.readGameLocationConfig(gameLocationConfig), 
            GameObjectReader.readGameAreaConfig(
                gameAreaMapConfig.get(gameLocationConfig)
        )
    );
  }
  
  Player player = new Player();
  GameRunner gameRunner = new GameRunner(player, gameAreaMap);

  // main game loop
  gameRunner.runGameLoop();
}
```

Some stuff from the GameRunner class
```java
public class GameRunner {

  public void runGameLoop() {
    while(isNotGameOver) {
      // print the description of the area if we've just arrived here
      if (isNewGameArea) {
        describeCurrentGameArea();
        this.isNewGameArea = false;
      }

      displayAvailablePlayerActions();
      PlayerAction nextAction = getNextPlayerAction();
      executeNextPlayerAction(nextAction)
    }
  }

  private void executeNextPlayerAction(PlayerAction action) {
    object[] actionArgs = action.getArgs();
    
    try {
      // This would be a good place to use the delegator pattern instead of a
      // switch statement, but it's honestly overkill for this small a project
      switch(action.getPlayerActionCode) {
        case TAKE_ITEM:
          // parse the arguments in the PlayerAction
          ItemHolder sender = (sender) actionArgs[0];
          ItemReference item = (ItemReference) actionArgs[1];
          Integer itemCount = (Integer) actionArgs[2];
          
          // perform the specific action
          transferItems(sender, this.player, item, itemCount)
          break;
        case GIVE_ITEM:
          // parse the arguments in the PlayerAction
          // perform the specific action
          break;
        case DESTROY_ITEM:
          // parse the arguments in the PlayerAction
          // perform the specific action
          break;
        case GOTO_NEXT_ROOM:
          // parse the arguments in the PlayerAction
          // perform the specific action
          break;
        // etc.
      }
    } catch (InvalidTypeCastException, IndexOutOfBoundsException) {
          throw new Exception(
              "Incorrect argument types provided for PlayerActionCode ${1}", 
              action.getPlayerActionCode()
          );
        }
  }
}
```
