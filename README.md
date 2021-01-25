	using System.Collections;
	using System.Collections.Generic;
	using UnityEngine;
	using UnityEngine.UI;

	public class Gameplay : MonoBehaviour
	{
    //<>

    //NOTE: THE POSITION OF THE NODES IN THE ARRAY WILL NOT BE THEIR POSITIONS IN THE REAL GRID AFTER THE GAME STARTS
    //USE THE POSITION VECTOR2 IN THE NODES ITSELF INSTEAD

    //NOTE 2: REMEMBER, ORIGIN IS TOP LEFT CORNER SO THE Y AXIS IN THE GRID IS REVERSED RELATIVE TO THE WORLD SPACE Y AXIS

    //Nodes matrix and scale value filled in by the level generator
    public Node[,] nodes;
    [HideInInspector]
    public float scale;


	public static Gameplay Instance;
	[SerializeField] private LevelRules rules;

	[SerializeField] private Slider[] starSliders;
	[SerializeField] private Image[] starImages;
	[SerializeField] private Image[] wonStars;
	[SerializeField] private GameOverPopup GameOverPopup;
	



	private int alreadyWonStars = 0;
	private int currentStars = 3;


    //Tracks the amount of moves made
    [HideInInspector]
    public int movecounter = 0;
    public Text movetext;
    public Text solvedtext;


	public int AlreadyWonStars
	{
		get { return alreadyWonStars; }
	}

	public LevelRules CurrentRules
	{
		get { return rules; }
	}

	private void Awake()
	{
		
		if (Instance != null && Instance != this)
		{
			Destroy(this.gameObject);
			return;
		}

		Instance = this;
	}

	private void Start()
	{
		if (LevelControl.Instance && LevelControl.Instance.currentLevelRules)
			rules = LevelControl.Instance.currentLevelRules;

		if(rules)
		{ 
			int slider1max = rules.TwoStarReflections;
			int slider2max = rules.OneStarReflections - rules.TwoStarReflections;

			SetSliderMaxValue(starSliders[0], slider1max);
			SetSliderMaxValue(starSliders[1], slider2max);

			SetSliderValue(starSliders[0], slider1max);
			SetSliderValue(starSliders[1], slider2max);

			//foreach (string tag in rules.requiredTags)
			//{
				//try
				//{
				//	GameObject[] objects = GameObject.FindGameObjectsWithTag(tag);
				//	foreach (GameObject obj in objects)
				//	{
					//	requiredObjects.Add(obj);
					//}
				//}
			//	catch(UnityException e)
			//	{
			//		Debug.LogWarning("GAMEPLAY: Tag " + tag + " is not defined. Check level rules: "+rules.name + " "+e);
			//	}
			}

			UpdateWonStars();
		}
	//}

	private void UpdateWonStars()
	{
		if (!PlayerPrefs.HasKey(rules.name + "_Stars")) return;

		alreadyWonStars = PlayerPrefs.GetInt(rules.name + "_Stars");
		for (int i=0; i < alreadyWonStars; i++)
		{
			wonStars[i].color = new Color(wonStars[i].color.r, wonStars[i].color.g, wonStars[i].color.b, 1.0f);
		}
	}

	//private bool CheckRequiredTags(GameObject obj, string[] tagList)
	//{
	//	foreach(string tag in tagList)
		//{
		//	if (obj.tag == tag)
		//		return true;
		//}
		//return false;
	//}

	private void Update()
	{
		UpdateSliders();
	}




	private void SetSliderMaxValue(Slider slider, int maxValue)
	{
		if (!slider) return;
		slider.maxValue = maxValue;
	}

	private void SetSliderValue(Slider slider, float value)
	{
		if (!slider) return;
		slider.value = value;
	}

	private void SetImageAlpha(Image img, float alpha)
	{
		if (!img) return;
		img.color = new Color(img.color.r, img.color.g, img.color.b, alpha);
	}


	private void UpdateSliders()
	{
		float slider1Value = starSliders[0].maxValue - movecounter;
		float slider2Value = starSliders[1].maxValue+ starSliders[0].maxValue - movecounter;

		SetSliderValue(starSliders[0], slider1Value);
		SetSliderValue(starSliders[1], slider2Value);

		if (slider1Value <= 0)
		{
			if(starImages[0].color.a > 0.3f) currentStars--;
			SetImageAlpha(starImages[0], 0.3f);

		}
		if (slider2Value <= 0)
		{
			if (starImages[1].color.a > 0.3f) currentStars--;
			SetImageAlpha(starImages[1], 0.3f);
		}
	}
		

    //Function to check if the puzzle has been solved
    public bool CheckSolve() {
        //Find start and finish nodes, return false if cannot find
        Node startNode = null;
        Node finishNode = null;
        foreach  (Node node in nodes) {
            if (node != null) {
                if (node.start) {
                    startNode = node;
                } else if (node.finish) {
                    finishNode = node;
					
                }
                if (startNode != null && finishNode != null) {
					
					break;

                }
            }
        }
        if (startNode == null || finishNode == null) {
			
            return false;
			
        }

        Node curNode = startNode;
        int curDir = 0;

        for (int i = 0; i < 4; i++) {
            if (startNode.connections[i]) {
                curDir = i + 1;
				
            }
        }
		

        //Main loop to repeat until either nowhere left to go or until the finish has been reached
        while (true) {
            Vector2Int directionVector = new Vector2Int(0, 0);
            int oppositeDir = 0;

            //Check if direction is out of grid
            switch (curDir) {
                case 1:
                    //Left
                    oppositeDir = 2;
                    if (curNode.gridPosition.x - 1 >= 0) {
                        directionVector = new Vector2Int(-1, 0);
                    }
                    break;
                case 2:
                    //Right
                    oppositeDir = 1;
                    if (curNode.gridPosition.x + 1 < nodes.GetLength(0)) {
                        directionVector = new Vector2Int(1, 0);
                    }
                    break;
                case 3:
                    //Up
                    oppositeDir = 4;
                    if (curNode.gridPosition.y - 1 >= 0) {
                        directionVector = new Vector2Int(0, -1);
                    }
                    break;
                case 4:
                    //Down
                    oppositeDir = 3;
                    if (curNode.gridPosition.y + 1 < nodes.GetLength(1)) {
                        directionVector = new Vector2Int(0, 1);
                    }
                    break;
            }
            if (directionVector.x != 0 || directionVector.y != 0) {
                //Asign next node
                curNode = FindNodeByPosition(curNode.gridPosition + directionVector);

                //Reset direction vector
                directionVector.x = 0;
                directionVector.y = 0;

                if (curNode != null) {
                    if (curNode.finish) {
                        for (int a = 0; a < 4; a++) {
                            if (a + 1 == oppositeDir) {
                                if (curNode.connections[a] == true) {
                                    return true;
                                }
                            }
                        }
                        return false;
                    } else {
                        //Assign new direction
                        bool dirFound = false;
                        bool canGo = true;
                        for (int a = 0; a < 4; a++) {
                            if (a + 1 == oppositeDir) {
                                if (curNode.connections[a] != true) {
                                    canGo = false;
                                }
                            }
                            if (a + 1 != oppositeDir && curNode.connections[a] == true) {
                                curDir = a + 1;
                                dirFound = true;
                            }
                        }
                        if (!dirFound || !canGo) {
                            break;
                        }
                    }
                } else {
                    break;
                }
            } else {
                break;
            }
        }

        return false;
    }

	private void WinLevel()
	{

		Debug.Log("You won.");
		StoreScore();

		if (!GameOverPopup) return;
		GameOverPopup.gameObject.SetActive(true);

		if (LevelControl.Instance && (LevelControl.Instance.CanGoToNextLevel(currentStars) || LevelControl.Instance.CanGoToNextLevel(alreadyWonStars)))
		{
			GameOverPopup.ShowGoodWin(currentStars);
			PlayerPrefs.SetString(("levellock" + (LevelControl.Instance.currentLevel + 1)), "unlocked");
		}
		else
		{
			GameOverPopup.ShowBadWin(currentStars);
		}
	}



	private void StoreScore()
	{
		if (currentStars > alreadyWonStars)
		{
			Debug.Log("storing current stars: " + currentStars);
			PlayerPrefs.SetInt((rules.name + "_Stars"), currentStars);
		}
	}

    //Function to check if node is able to move in the given direction
    public bool CheckIfMovable(Node node, int direction) {
        Vector2Int directionVector = new Vector2Int(0, 0);
        switch (direction) {
            case 1:
                //Left
                if (node.gridPosition.x - 1 >= 0) {
                    directionVector = new Vector2Int(-1, 0);
                }
                break;
            case 2:
                //Right
                if (node.gridPosition.x + 1 < nodes.GetLength(0)) {
                    directionVector = new Vector2Int(1, 0);
                }
                break;
            case 3:
                //Up
                if (node.gridPosition.y - 1 >= 0) {
                    directionVector = new Vector2Int(0, -1);
                }
                break;
            case 4:
                //Down
                if (node.gridPosition.y + 1 < nodes.GetLength(1)) {
                    directionVector = new Vector2Int(0, 1);
                }
                break;
        }
        if (directionVector.x == 0 && directionVector.y == 0) {
            return false;
        } else if(FindNodeByPosition(node.gridPosition + directionVector) == null) {
            return true;
        } else {
            return false;
        }

    }

    //Function to move a node, requires a node (node to move) and an int (1- left 2- right 3- up 4- down)
    //Repeats the checkifmovable function a bit, this is done because the purpose is slightly different
    //And there's no need to repeat the switch statement twice
    public void MoveNode(Node node, int direction) {
        Debug.Log("Moving node in direction: " + direction.ToString());

        //Check if movement can be done
        bool moveable = false;
        Vector2Int directionVector = new Vector2Int(0,0);
        switch (direction) {
            case 1:
                //Left
                if (node.gridPosition.x - 1 >= 0) {
                    moveable = true;
                    directionVector = new Vector2Int(-1,0);
                } else {
                    Debug.Log("Movement out of grid");
                }
                break;
            case 2:
                //Right
                if (node.gridPosition.x + 1 < nodes.GetLength(0)) {
                    moveable = true;
                    directionVector = new Vector2Int(1, 0);
                } else {
                    Debug.Log("Movement out of grid");
                }
                break;
            case 3:
                //Up
                if (node.gridPosition.y - 1 >= 0) {
                    moveable = true;
                    directionVector = new Vector2Int(0, -1);
                } else {
                    Debug.Log("Movement out of grid");
                }
                break;
            case 4:
                //Down
                if (node.gridPosition.y + 1 < nodes.GetLength(1)) {
                    moveable = true;
                    directionVector = new Vector2Int(0, 1);
                } else {
                    Debug.Log("Movement out of grid");
                }
                break;
        }
        if (moveable) {
            if (FindNodeByPosition(node.gridPosition + directionVector) == null) {
                node.Move(directionVector,scale);
                movecounter++;
                Debug.Log("Moving node");

                if (movetext != null) {
                    movetext.text = "Moves: " + movecounter.ToString();
                }

                //Check if the puzzle has been solved
                if (CheckSolve()) {
                    //TO-DO: SOLVED
                    Debug.Log("Solved");
                    if (solvedtext != null) {
                        solvedtext.text = "Solved";
                        solvedtext.color = Color.green;
						

                    }
                } else {
                    if (solvedtext != null) {
                        solvedtext.text = "Not Solved";
                        solvedtext.color = Color.red;
                    }
                }
            } else {
                Debug.Log("Movement blocked");
            }
        }

    }

    public Node FindNodeByPosition(Vector2Int position) {
        foreach (Node node in nodes) {
            if (node != null) {
                if (node.gridPosition == position) {
                    return node;
                }
            }
        }
        return null;
    }

    }

