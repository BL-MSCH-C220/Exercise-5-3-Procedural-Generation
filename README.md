# Exercise 5.3-Procedural Generation

Exercise for MSCH-C220

This exercise is designed to give you some brief experience with creating a procedurally-generated 3D maze. It is based on the excellent tutorial by [KidsCanCode](https://kidscancode.org/blog/2018/08/godot3_procgen1/).

Fork this repository. When that process has completed, make sure that the top of the repository reads [your username]/Exercise-5-3-Procedural-Generation. *Edit the LICENSE and replace BL-MSCH-C220 with your full name.* Commit your changes.

Clone the repository to a Local Path on your computer.

Open Godot. Import the project.godot file and open the "Procedural Generation" project.

The first step is to set up the tiles we will use to generate the maze. In the File System panel, you will see that I have created the following scenes for you:
 * res://Maze/Tile0.tscn
 * res://Maze/Tile1.tscn
 * res://Maze/Tile2.tscn
 * res://Maze/Tile3.tscn
 * res://Maze/Tile4.tscn
 * res://Maze/Tile5.tscn
 * res://Maze/Tile6.tscn
 * res://Maze/Tile7.tscn
 * res://Maze/Tile8.tscn
 * res://Maze/Tile9.tscn
 * res://Maze/Tile10.tscn
 * res://Maze/Tile11.tscn
 * res://Maze/Tile12.tscn
 * res://Maze/Tile13.tscn
 * res://Maze/Tile14.tscn
 * res://Maze/Tile15.tscn

Next you will need to perform some surgery on these newly-created scenes. The algorithm relies on the existence of "Tiles" with all permutations of possible walls: N, E, S, and W. Remove walls (and corresponding collision shapes for the following tiles):
 * Tile0: All walls removed (you should only have Tile, StaticBody, Ground, and Ground_Collision nodes left)
 * Tile1: Only N remains
 * Tile2: Only E
 * Tile3: N and E
 * Tile4: Only S
 * Tile5: N and S
 * Tile6: E and S
 * Tile7: N, E, and S
 * Tile8: Only W
 * Tile9: N and W
 * Tile10: E and W
 * Tile11: N, E, and W
 * Tile12: S and W
 * Tile13: N, S, and W
 * Tile14: E, S, and W
 * Tile15: N, E, S, and W


Now open res://Maze/Maze.gd. Replace the contents of the script with the following:
```
extends Node3D

const N = 1 					# binary 0001
const E = 2 					# binary 0010
const S = 4 					# binary 0100
const W = 8 					# binary 1000

var cell_walls = {
	Vector2(0, -1): N, 			# cardinal directions for NESW
	Vector2(1, 0): E,
	Vector2(0, 1): S, 
	Vector2(-1, 0): W
}

var map = []

var tiles = [
	preload("res://Maze/Tile0.tscn")	# all the tiles we created
	,preload("res://Maze/Tile1.tscn")
	,preload("res://Maze/Tile2.tscn")
	,preload("res://Maze/Tile3.tscn")
	,preload("res://Maze/Tile4.tscn")
	,preload("res://Maze/Tile5.tscn")
	,preload("res://Maze/Tile6.tscn")
	,preload("res://Maze/Tile7.tscn")
	,preload("res://Maze/Tile8.tscn")
	,preload("res://Maze/Tile9.tscn")
	,preload("res://Maze/Tile10.tscn")
	,preload("res://Maze/Tile11.tscn")
	,preload("res://Maze/Tile12.tscn")
	,preload("res://Maze/Tile13.tscn")
	,preload("res://Maze/Tile14.tscn")
	,preload("res://Maze/Tile15.tscn")
]

var tile_size = 2 						# 2-meter tiles
var width = 20  						# width of map (in tiles)
var height = 12  						# height of map (in tiles)

func _ready():
	randomize()
	make_maze()
	
func check_neighbors(cell, unvisited):
	# returns an array of cell's unvisited neighbors
	var list = []
	for n in cell_walls.keys():
		if cell + n in unvisited:
			list.append(cell + n)
	return list
	
func make_maze():
	var unvisited = []  # array of unvisited tiles
	var stack = []
	# fill the map with solid tiles
	for x in range(width):
		map.append([])
		map[x].resize(height)
		for y in range(height):
			unvisited.append(Vector2(x, y))
			map[x][y] = N|E|S|W 		# 15
	var current = Vector2(0, 0)
	unvisited.erase(current)
	while unvisited:
		var neighbors = check_neighbors(current, unvisited)
		if neighbors.size() > 0:
			var next = neighbors[randi() % neighbors.size()]
			stack.append(current)
			var dir = next - current
			var current_walls = map[current.x][current.y] - cell_walls[dir]
			var next_walls = map[next.x][next.y] - cell_walls[-dir]
			map[current.x][current.y] = current_walls
			map[next.x][next.y] = next_walls
			current = next
			unvisited.erase(current)
		elif stack:
			current = stack.pop_back()
	for x in range(width):
		for z in range(height):
			var tile = tiles[map[x][z]].instantiate()
			tile.position = Vector3(x*tile_size,0,z*tile_size)
			tile.name = "Tile_" + str(x) + "_" + str(z)
			add_child(tile)
```

As described in the KidsCanCode tutorial and in the demonstration video, this script uses a Recursive Backtracker algorithm to produce a fully-connected randomized maze. Run the res://Maze/Maze.tscn scene and see the resulting maze. The contents of Maze.gd would be a great snippet to add to your gists.

Next, you will need to add an in-game menu to the game.

Replace the contents of res://Global.gd with the following:
```
extends Node

func _ready():
	process_mode = PROCESS_MODE_ALWAYS		# global should never be paused

func _unhandled_input(event):
	if event.is_action_pressed("Menu"):	# instead of quitting, show the menu
		var menu = get_node_or_null("/root/Game/Menu")
		if menu == null:
			get_tree().quit()
		else:
			if not menu.visible:
				menu.show()
				get_tree().paused = true 	# pause the game while the menu is visible
			else:
				menu.hide()
				get_tree().paused = false
```

Then add the following functionality to res://UI/Menu.gd:
```
extends Control

func _on_Restart_pressed():				# if we restart, then unpause the game and the reload the scene
	get_tree().paused = false
	get_tree().change_scene_to_file("res://Maze/Maze.tscn")

func _on_Quit_pressed():
	get_tree().quit()
```

Finally, you will need to adjust the menu so it continues to work even if the game is paused. Select the Menu node in the Scene Panel. In the Inspector Panel->Node, change Process Mode to "Always".

Test the project. You should be able to pause the game (by hitting escape), restart, and quit.

Quit Godot. In GitHub desktop, add a summary message, commit your changes and push them back to GitHub. If you return to and refresh your GitHub repository page, you should now see your updated files with the time when they were changed.

Now edit the README.md file. When you have finished editing, commit your changes, and then turn in the URL of the main repository page (https://github.com/[username]/Exercise-5-3-Procedural-Generation) on Canvas.

The final state of the file should be as follows (replacing the "Created by" information with your name):
```
# Exercise 5.3—Procedural Generation

Exercise for MSCH-C220

An implementation of a procedurally-generated 3D maze. Also, a simple in-game menu.

## Implementation

Built using Godot 4.1.1

## References

This project is an adaptation of the excellent tutorial from [KidsCanCode](https://kidscancode.org/blog/2018/08/godot3_procgen1/)

## Future Development

None

## Created by 

Jason Francis
```
