# Football Pitch Reconstruction from Match Images 

This is a computer vision project I'm working on to see how much information about a football match can be reconstructed from a normal broadcast image.

The main idea is pretty simple: I start from an image of a football match and I want to detect the players, understand which team they belong to, figure out where they actually are on the pitch, and finally recreate the scene as a sort of 2D top-down representation of the field.

So basically:

Broadcast image then Detect players then Identify teams then Understand the pitch perspective then Map players to their coordinates on pitch then Build 2D representation

The final result should look something like having the original image on one side and a simplified pitch on the other, with dots representing the detected players in their estimated real positions.

## This is what I plan to do: 
### 1. Data selection

For main dataset I'm currently using SoccerNet, specifically the football broadcast images and the corresponding camera annotations. 
Each observation basically is: 
- jpg which is the actual broadcast match image 
- json containing the annotations for the pitch elements visible in that image (sidelines, penalty area, goal posts, etc)

Before deciding exactly how to use these I just started doing some exploratory analysis in the dataset and understand the notation and what they represent.

# Progress So Far 
So far what I did is:
- loaded the data and visualized some instances of those jpg / jsons pairs 
- projected the annotations coordinates on the original image to check if they match (they mostly do, checked some random instances)
- generated binary masks from the json, basically created a black and white image with in white just the annotation parts (like black pitch with white lines for every annotation)
- analyzed the annotations kinds, how many, their distributions etc 
The masking I think could be a firt way of understanding how the Json annotations could be transformed into a target that a computer vision model can learn from. 


### 1. Player detection (object detection)
The next main step is detecting all the visible players in the image. I'll probably start from an existing object detection model and fine-tune it on football images rather than training something completely from scratch.

The main output here will be the bounding box of every visible player (and potentially referees/goalkeepers as separate classes).

### 2. Team identification

Once the players are detected, I need to figure out which team each one belongs to.

My initial idea is to use the colour of the kits rather than manually defining every possible team. For example, I could extract colour information from each player's crop and use clustering to automatically separate the players into the two teams.

Goalkeepers and referees make this slightly more complicated, so I'll see what to do with that later.

### 3. Pitch detection 

I need to understand the perspective of the camera so that I can translate an image position into an actual position on the football pitch.

Probably need to detect recognizable parts of the pitch like:

- penalty box corners
- halfway line
- touchlines
- centre circle
- other lines 

These give me reference points whose real positions on a football pitch are known.

### 4. Perspective transformation

Using the pitch reference points, I can estimate a **homography** between the broadcast camera view and a top-down football pitch.

This should let me take something like:

`player feet = (820, 440) pixels`

and transform it into something like:

`player position = (61.3 m, 24.7 m)`

on the actual pitch.

### 5. Reconstruct the scene

Once I have the estimated pitch coordinates and team of every visible player, I can finally generate the actual 2D representation.

Something roughly like:

```text
┌────────────────────────────────────────────┐
│           🔵                               │
│                    🔴                      │
│     🔴          🔵                         │
│─────────────────────⚽─────────────────────│
│              🔵              🔴            │
│                       🔴                   │
└────────────────────────────────────────────┘
```


The underlying output should also be structured data containing things like player ID, team and estimated `(x, y)` pitch coordinates.

## Ball detection?

Maybe

The ball is tiny compared to players and can easily be blurred, hidden or difficult to distinguish, so initially I'll focus on getting the player reconstruction working properly.

## Evaluation

I'll evaluate the different parts separately:

- player detection performance
- team classification accuracy
- pitch/keypoint detection error
- homography / projection accuracy
- final player-position error on the reconstructed pitch

I'm interested in seeing where the errors in the final reconstruction actually come from, like whether inaccurate player detection matters much less than small errors in the estimated camera perspective.


# Possible extension: real football vs EA Sports FC

Something I'd really like to explore after getting the main project working is whether the same system can work on EA Sports FC gameplay.

The idea would be to take screenshots from FC (probably FC 27 by the time I get to this part) and pass them through the same or adapted reconstruction pipeline.

Apart from just being a fun extension, I think this could turn into an interesting small experiment about the difference between real and synthetic football images.

For example, I could investigate whether models trained on real broadcast football generalize to FC gameplay and, if they don't, where the differences actually appear:

- player detection

- pitch-line detection

- team/kit identification

- camera calibration

- final position reconstruction
Although the harder part here might be the annotation thing cause with game images either I actually manually do them for a subset of instances or I don't use them but I don't know yet. 

Ideally I could also find real and game images with somewhat similar camera perspectives or situations and compare the intermediate representations produced by the models.

The broader question would basically be:

How differently does a computer vision system perceive a real football broadcast and a visually similar representation of football inside a videogame?

Maybe there will be a clear domain gap, maybe some components will transfer surprisingly well, or maybe the game will actually be easier because the environment is visually cleaner and more controlled.

For now though, this is an extension. The first goal is still:

Take a normal real football broadcast image and automatically turn it into a reasonably accurate top-down representation of the visible players on the pitch.


This project uses SoccerNet data. Due to the dataset's access restrictions, the original images and annotations are not included in this repository. Access should be requested directly through SoccerNet.
