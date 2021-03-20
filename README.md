There are two folders in this repository, each with a complete Godot project.

Open Godot. Navigate to the 01-Config folder. Import the project.godot file and open the "Save and Load: Config" project.

This project implements save an load using a config file. This is certainly an easy way to save data, but it has some important limitations (which is why it should only be used to save configuration information).

Open Global.gd, and replace the `save_game()` and `load_game()` methods with the following (starting on line 58):
```
func save_game():
	save_data["general"]["coins"] = []
	save_data["general"]["mines"] = []
	for c in Coins.get_children():
		save_data["general"]["coins"].append(c.position)
	for m in Mines.get_children():
		save_data["general"]["mines"].append(m.position)
	for section in save_data.keys():
		for key in save_data[section]:
			save_file.set_value(section, key, save_data[section][key])
	save_file.save(SAVE_PATH)

func load_game():
	var error = save_file.load(SAVE_PATH)
	if error != OK:
		print("Failed loading file")
		return
	
	save_data["general"]["coins"] = []
	save_data["general"]["mines"] = []
	for section in save_data.keys():
		for key in save_data[section]:
			save_data[section][key] = save_file.get_value(section, key, null)
	var _scene = get_tree().change_scene_to(Game)
	call_deferred("restart_level")
```

Save your project and test it. Blow up a few mines and collect a few coins. Press escape to access the menu, and then save your game. Press the load button and watch what happens. Repeat a few times.

Now, look at the files again in the Windows Explorer or the System Finder. You should see a new file in the 01 Config folder: settings.cfg. Open that file with some kind of text editor (you could use VS Code) and take a look at it. What would happen if you were to edit this file directly?

In the file, change both the score and lives values to 1000. Save your changes and close the file. Go back to Godot and run your project; load your game. What happened?

Now, it's time to do it the right way. Open the 02-Save folder and import the Godot project. Open Global.gd in the Save and Load: Save File project, and replace the `save_game()` and `load_game()` methods (line 58) with the following:
```
func save_game():
	save_data["general"]["coins"] = []
	save_data["general"]["mines"] = []
	for c in Coins.get_children():
		save_data["general"]["coins"].append(var2str(c.position))
	for m in Mines.get_children():
		save_data["general"]["mines"].append(var2str(m.position))

	var save_game = File.new()
	save_game.open_encrypted_with_pass(SAVE_PATH, File.WRITE, SECRET)
	save_game.store_string(to_json(save_data))
	save_game.close()
	
func load_game():
	var save_game = File.new()
	if not save_game.file_exists(SAVE_PATH):
		return
	save_game.open_encrypted_with_pass(SAVE_PATH, File.READ, SECRET)
	var contents = save_game.get_as_text()
	var result_json = JSON.parse(contents)
	if result_json.error == OK:
		save_data = result_json.result
	else:
		print("Error: ", result_json.error)
	save_game.close()
	
	var _scene = get_tree().change_scene_to(Game)
	call_deferred("restart_level")
```

Now, instead of using a config file, we are using an actual save file in user space (instead of in the application itself). Also, we are encrypting the file using the SECRET passphrase. Depending on your operating system, that file is stored in one of the following locations:
```
Windows: C:\Users\<username>\AppData\Roaming\Godot\app_userdata\<project>\
MacOS: ~/Library/Application\ Support/Godot/app_userdata/_APPLICATION_NAME_
Unix: ~/.local/share/godot/app_userdata/_application_name_/
```

Test the game and play with the save and load functionality.
```
# Exercise-04d-Save-and-Load
Exercise for MSCH-C220, 15 March 2021

The fourth exercise for the 2D Platformer project, exploring save and load (in two projects).

## Implementation
Built using Godot 3.2.3

The player sprite is an adaptation of [MV Platformer Male](https://opengameart.org/content/mv-platformer-male-32x64) by MoikMellah. CC0 Licensed.

The coin sprite is provided by Kenney.nl: [https://kenney.nl/assets/puzzle-pack-2](https://kenney.nl/assets/puzzle-pack-2).

The explosion animation is also adapted from Kenney.nl: [https://kenney.nl/assets/tanks](https://kenney.nl/assets/tanks).

## References
For more information about save and load in Godot, visit the Godot documentation: [https://docs.godotengine.org/en/stable/tutorials/io/saving_games.html#saving-and-reading-data](https://docs.godotengine.org/en/stable/tutorials/io/saving_games.html#saving-and-reading-data)

## Future Development
None

## Created by 
Gwen Hall
