# Dark Sun Scripter's Manual

If you are scripting for us, you should also be familiar with the [player's manual](playersmanual.md), the [dungeon master's manual](dmmanual.md) and [builder's manual](buildersmanual.md).  Each of these publications build on each other.  Everything in those manuals will ensure you're familiar with the systems present in this module and how they are used.  This manual will specifically introduce you to the expected scripting style as well as some of the advanced features of the framework and other systems.

* [Expectations](#expectations)
* [Data Management](#data-management)
* [Systems](#systems)
    * [Core Framework](#core-framework)
    * [HCR2](#hcr2)
    * [DMFI](#dmfi)

## Expectations

As a volunteer project, we fully expect that any person that works on this project will not be around forever.  With that in mind, we expect everyone that contributes to the project to adhere to specific customs and styles so that future contributors can easily read the code and understand what is going on.  Here's a list of general expectations for scripting:

* All functions, including internal function, will be prototyped and documented.  We do not expect, nor desire, comments for every line of code, but we do expect that the function description associated with the function prototype give the next programmer enough information to know what's going on.  Here's an example of bad prototyping and documentation:

    ```

    ```

    Yep, there's nothing there.  That's bad.  Here's an example of good prototyping and documentation.  This is a simple procedures, so not much is required.  For more complicated procedures, you either need more documentation, or you need to break the procedure up.

    ```c
    // ---< ActivatePlugin >---
    // ---< core_i_framework >---
    // Runs oPlugin's OnPluginActivate script and sets its status to ON. Returns
    // whether the activation was successful. If bForce is TRUE, will activate the
    // plugin even if its status is already ON.
    int ActivatePlugin(object oPlugin, int bForce = FALSE);
    ```

* All functions will contain debugging and logging messages, as required, to aid in future changes/debugging and to inform the module ownership, DMs and players (as appropriate) what is going on.  We have several tools to accomplish this easily.  See [debug messaging](#debug-messaging) and [module communication](#module-communication).  Here's an example of the debug messaging available in the module:

    ``` c
    Debug("Successfully created encounter with ID " + sEncounterID);
    ```

    ``` c
    Warning("Cannot activate plugin '" + sPlugin + "': denied");
    ```

    This message will be sent to all destinations as opted in `core_c_config`.  You are not necessarily limited to basic data message, though.  Here's an example of a more complicated Debug message that will help everyone determine exactly what is happening within a system:

    ``` c
    Debug("Checking " + sEvent + " script " + IntToString(i + 1) + sCount +
        "\n    Script: " + sScript +
        "\n    Priority: " + PriorityToString(fPriority) +
        "\n    Source: " + GetName(oSource));
    ```

    Although there is nothing wrong with simple debug messages, ensure there is enough information available to allow any scripter to determine where the issues may lie.  
    *Note: **We DO NOT send debug messages with the SendMessageToPC() function.***

* Data (primarily variables) intended for the module or any player will be handled by our [data handling functions](#data-handling) instead of setting variables on the Module or on the PC.

* Scripting is intended to make the builder's life easier.  Where feasible, use wrapper functions instead of requiring the builder to know every possible variable or option they can send to a function.  The debug system is a great example of this.  The `Debug()` function handles the heavy liftin go sending messages where they need to go, but it has three wrapper/alias functions:  `Warning()`, `Error()`, and `CriticalError()`.  Each one sets specific variables to allow coloring of the variaous messages, but all the builder/scripter has to know if the function name.

## Data Management System

Saving variables and determining who a player is are two of the most common functions in nwscript.  To that end, we're provided alias functions for them to prevent overloading the Module and PC objects, and to quickly determine the identify of a PC.  Just about every script in the module should have an include reference to `dsutil_i_data`.  This script itself includes [`util_i_debug`](../framework/src/utils/util_i_debug.nss) and [`dsutil_i_comms`](../utilities/dsutil_i_comms), so including `dsutil_i_data` should provide you with a great portion of the functionality to set variables, determine identities, provide debug information and send messages around the module.

Here are the basic functions it provides:
* `_GetIsPC()` - a replacement for nwscript's `GetIsPC()`.  Our version determines whether the character is player-controlled (PC) and not a DM.  So if you're trying to determine if a player is a PC and not a DM, use this function.
* `_GetIsDM()` - a replacement for nwscript's `GetIsDM()`.  Our version determines whether the passes character object is a DM or a DM possessing an NPC.
* `_GetIsPartyMember()` - will return whether the first passed object is a party member of the second passed object.

* `[_Get, _Set, _Delete]Local*` - module-specific replacements for `[get,set,delete]Local*`.  For now, these are just wrappers, however, they will be used later for object/variable inheritance and other functions, so best to start using them now.  To set a variable on the module, the object `MODULE` should be passed:  `_SetLocalInt(MODULE, sVarName, nValue)`.  Also allows us to delete the HCR2 methodology of using separate functions to set module variables and player-persistent variables.  Just call our version and it'll figure it our for you.  If none of the special conditions are met (i.e. for PCs, inheritance, GetModule(), etc.) it just processes the variable on the passed object as the normal Bioware function would.

To learn more and understand exactly how the functions work, open up [`dsutil_i_data`](../utilities/dsutil_i_data.nss) and take a look!

*Acknowledgement:  As acknowledged in our [acknowledgements document](acknowledgements.md/#hcr2), many of these functions are taken directly from Edward Beck's HCR2 system and modified for our use.*

## Framework System

The entire module rests on the core framework developed, maintained and continuously improved by Michael Sinclair (squattingmonk).  The framework does an amazing job of organizing code and managing events, as well as providing access to efficient list management, debugging utilities, datapoints, text coloring, database interface and script library functions.  This is all done inside of nwscript with the exception of the database interface, which uses NWNXEE. I cannot say enough about how well this framework handles the basic functionality of the module.  In order to successfully script here, you must understand how the framework works and the various methods to call functions, events, etc.

***Note:  No pull requests will be accepted that involve modification to any file in the `framework` submodule.***

* [Math](#math-functions)
    * [DS Math](#ds-math-functions)
* [Lists](#list-functions)
    * [DS Lists](#ds-list-functions)
* [Datapoints](#datapoint-functions)
* [Debugging](#debug-functions)
* [Color](#coloring-functions)
* [Library](#library-functions)
* [Event Management](#event-management)

#### Math Functions

[`util_i_math`](../framework/src/utils/util_i_math) provides access to some useful basic math functions such as mod, min, max, clamps, etc.  None of these require detailed explanation.  Open up the file and take a look at what's available.

###### DS Math Functions

[`dsutil_i_math`](../utilities/dsutil_i_math) is a supplement to the framework's `util_i_math` and provides a rounding function for floats.

#### List Functions

The framework provides access to two types of lists:  comma separated values (CSVs) and array-like lists (varlists).  There is not extensive documentation here because the author has provided his own [list documentation](../framework/docs/lists.md).  In addition to the functions and ideas found there, there is another script file called `util_i_lists` which contains functions to swap from one list type to the other.  These lists are extensively used throughout the module.

An example of the power of these lists is the way in which languages are added into the DMFI language system.  When initiated, the language system searches for all items in a CSV, which are references to game objects (items) that contain the appropriate variables to allow language translation.  As each object is found, it is added to an object varlist on the DMFI data point.  Simultaneously, a CSV is created with the names of each of the languages.  This allows a quick way to create an index of languages.  The CSV is searched by language name to determine its index, then the object list is refernced by index to get the required object and its variables.  In this way, new languages can be added to the module without any scripting at all.  A new language initializer items simply needs to be created with the appropriate variables and the expected tag.

###### DS List Functions

[`dsutil_i_varlist`](../utilities/dsutil_i_varlist) is a supplement to the framework's `util_i_varlists` and provides the ability to manipulate pseudo-vector variables.

#### Datapoint Functions

Much like the list functions above, the author has provided his own [datapoint documentation](../framework/docs/datapoints.md).  Datapoints are used extensively by the framework as well os other module systems, such as DMFI, travel area encounters, etc.  Using datapoints allows us to provide module-wide access to various variables without overloading the module object.

#### Debugging Functions

The author has provided his own [debugging documentation](../framework/docs/debugging.md).  In addition to debugging functions, we also have systems for module-wide communications and logging.

#### Coloring Functions

Another utility, `util_i_color`, provides function for coloring various strings for presentation to players.  Many of the functions in this script are dependent on `util_i_math`.  These functions can be sued to color strings for debugging presentation, messaging around the module and within custom conversations.  Your primary use will probably be `HexColorString()` which allows you to use one of the many pre-defined colors within the script.

#### Library Functions

//TODO - this is not very explanatory.  Rewrite.

The author has included his own [library documentation](../framework/docs/libraries.md) to help in understanding how the library system works.  The library system allows the framework's [event management system](#event-management) to do what it does so well.  The gist of the system, however, is that you expose the functions you wish to be public through the library and pointers to those functions are stored in varlists on various datapoints in the module.  When any of those functions is called from any part of the module, the library system is able to find the function and run it, without the scripter/builder having to know which library it's in.  This does not mean you can call a function from another script directly without an `#include` reference, but it does mean that you can run an event script (such as `OnAreaExit`) that resides in the HCR2 fugue system from any and all areas without having to point the event directly to the script.

Here are some notes from our implementation of the library system:
* All scripts associated with a library, if written correctly, become part of one large script at compile time.  This is the primary reason each our libraries has only six different scripts in them (configuration, text, constants, events, main, and plugin/library).
* Bioware override scripts can be housed in any library and don't need to have the same name as the script they're overriding, only the event name passed through the library does.  Here's an example of a library exposure that override Bioware's nw_s2_animalcom:

    ``` c
    void OnLibraryLoad()
    {
        RegisterLibraryScript("nw_s2_animalcom", 1);
    }

    void OnLibraryScript(string sScript, int nEntry)
    {
        switch (nEntry)
        {
            case 1:  MyAnimalCompanionScript(); break;
            default: CriticalError("Library function " + sScript + " not found");
        }
    }
    ```

* The same can be done with item-based scripting.  For both cases, separate, specially named scripts are not required, just expose your function to the library system with the correct name and you can run any function or script that you want.  Here's an example of tag-based scripting from HCR2's torch subsystem.  This isn't the entire script, it just shows the portion necessary to understand tag-based scripting.  The actual name of the items are carried inside constants (H2_LANTERN and H2_OILFLASK, in this case), so you can change the tag of the item and not have the system break.  When any player uses an item tagged as a lantern (`h2_lantern`), the system looks for a script with that item name, which matches the constant we passed (`H2_LANTERN = "h2_lantern"`).  Once found, it looks into the library for the nEntry that matches the one we assigned (`2` in this case).  

    ``` c
    void OnLibraryLoad()
    {
        ...

        // ----- Tag-based Scripting -----
        RegisterLibraryScript(H2_LANTERN,            2);
        RegisterLibraryScript(H2_OILFLASK,           3);

        ...
    }

    void OnLibraryScript(string sScript, int nEntry)
    {
        switch (nEntry)
        {
            ...

            // ----- Tag-based Scripting -----
            case 2: torch_lantern();       break;
            case 3: torch_oilflask();      break;

            ...
        }
    }
    ```

    It then runs the script associated with that entry (`torch_lantern()`), which contains all the scripting you would normally keep in its own .nss, without having to maintain another file.

    ``` c
    void torch_lantern()
    {
        int nEvent = GetUserDefinedItemEventNumber();

        // * This code runs when the item is equipped
        // * Note that this event fires PCs only
        if (nEvent ==  X2_ITEM_EVENT_EQUIP)
        {
            h2_EquippedLightSource(FALSE);
        }
        // * This code runs when the item is unequipped
        // * Note that this event fires for PCs only
        else if (nEvent == X2_ITEM_EVENT_UNEQUIP)
        {
            h2_UnEquipLightSource(FALSE);
        }
    }
    ```
    
* 
* 
* 
* 





#### Event Management




## Dialog/Conversation System


## Quest Management System


## Travel Encounter System

The travel encounter system is easily expandable and requires no scripting knowledge to make work.  The travel encounter system is designed to be used on inter-city travel maps. There are two areas required to make the encounter system work.

1. **The travel area.**  The travel area is usually a very large area (32x32 is the max) that is used for intra-city travel.  To mark a travel area as an area that can spawn encounters, you must set a variable on the travel area.  The value of this variable is a comma delimited list of the resrefs of all the encounter areas you want to use.  It must contain at least one if you want this travel area to spawn encounters, but there is no limit to how many areas can be assigned.  When an encounter is spawned, the encounter area will be chosen at random from this list.

    |Variable Name|Type|Value|
    |---|---|---|
    |TRAVEL_ENCOUNTER_AREAS|string|area1_resref,area2_resref|

2. **The encounter area(s).**  A travel area can have several encounter areas associated with it.  For example, if builders have created multiple encounter areas, you can select which ones you want to use as your encounter areas based on their tileset matching the general theme of the travel area.  A successful encounter area will have the following features:

    * 1 waypoint designated as the primary portal into the encounter area
    * 2-5 waypoints designated as secondary portals into the encounter area
    * 2-5 waypoints designated as spawn points for the encounter enemies
    * 1-2 exits that are obvious to the player and require the characters to traverse spawn waypoints to reach them

    Although the encounter area itself requires no specific variables, each waypoint added must have one variable assigned to it in order for it to be used successfully by the travel encounter system.  Below are the three possible values.  Only one should be assigned.  The primary waypoint will be the waypoint the PC triggering the encounter will port to, along with any party members within a specific distance of the PC.  There should be only one primary waypoint designated.  If two are designated, the first one the system finds will be marked primary, the rest will be ignored.  If no waypoints are designated as primary, the encounter will not occur.  Any party members outside the specified distance, but still close to the triggering PC will be ported to one of the secondary waypoints at random.  There is no limit to the number of secondary waypoints you can place.  If there are no secondary waypoints placed, the primary waypoint will be used.  The encounter enemies will spawn at randomly selected spawn waypoint.

    |Variable Name|Type|Value|
    |---|---|---|
    |TRAVEL_WAYPOINT_TYPE|string|primary|
    |TRAVEL_WAYPOINT_TYPE|string|secondary|
    |TRAVEL_WAYPOINT_TYPE|string|spawn|

    *Note: no triggers or other objects in the encounter area require specific variables for this system to function.*

That's it!  If all of the above conditions are satisfied, the encounters will happen automatically.

## Dungeon Master Friendly Initiative

#### Languages

Languages are initialized through language initalizer items in the module.  The system is designed to allow builder to add, modify or remove languages at any time.

To add a new language to the module, the following requirements must be met:

* A language initializer item must be created within the toolset.  Make a copy of the template language initializer item to accomplish this.  It already has the required variables set on it, you just need to change the variables to match your new language.  Once again, do not edit the template, edit a copy of the template

* The language initializer item must be named `dmfi_l_<language name>`.  The language name cannot be more than nine characters long due to the 16 character limit in NWN.  Do not use spaces in your language name or the item name.

* The following variables must be set on the language initializer item:

    |Variable Name|Type|Value|
    |---|---|---|






## Questions

If you have any questions about installing these tools, and you're sure you followed the instructions above correctly, you're best bet is to get onto the [Dark Sun discord](https://discordapp.com/channels/468225176773984256/468225176773984258) and ask a question about installing the tools.  If you're not a member of our discord, you can [join](https://discord.gg/8ZxgMRc).  If you tag me (@tinygiant) in your post, I'll likely answer pretty quickly.  If you don't tag me, I may not see the question at all, but one of our many other team members might be able to help.  I'm happy to answer discord DMs also if you don't want to join the discord.

## Conclusion

I know this was a lot of information, but, again, it all boils down to just a few commands once you're used to it.  The majority of this tutorial is aimed at our audience that is new to this process.  If you've decided to use VS Code, which can accomplish a lot of these steps in a visual environment, please read the [~~VS Code Tutorial~~coming soon](vscode.md).  This is not really necessary for most content developers, but if you're a scripter, I highly recommend your consider VS Code as your primary development environment for this module.