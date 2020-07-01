- [Lua Coding Guidelines](#lua-coding-guidelines)
	- [But why though?](#but-why-though)
	- [Isn’t this going to make it harder for newcomers?](#isnt-this-going-to-make-it-harder-for-newcomers)
	- [Visual Studio Code](#visual-studio-code)
	- [Variables](#variables)
	- [Functions](#functions)
	- [Conditional Statements](#conditional-statements)
	- [Tables](#tables)
	- [Loops](#loops)
	- [Classes](#classes)

# Lua Coding Guidelines
## But why though?
1. Having a consistent coding style makes it easier for everybody to read each others code.
2. Following the guidelines here makes reading the code and following its logic easier in general, because variables will have descriptive names, functions will be short and have little nesting.

## Isn’t this going to make it harder for newcomers?
No. Newcomers to programming don’t have a sense for clean and readable code. They have no experience with refactoring.
When following coding guidelines they will learn to write somewhat better code from the beginning.

## Visual Studio Code
- Lua files should be edited with VS Code or an editor that supports EmmyLua
- Install the following VS Code extensions:
	- For linting and intellisense:
		- [Lua - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=sumneko.lua)
	- For formatting:
		- [vscode-lua-format - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=Koihik.vscode-lua-format)


## Variables
- Declare variables as `local` when possible.
- Variables must be defined in `snake_case`

BAD:

```lua
local mynumber = 1
local myNumber = 1
local MyNumber = 1
local MYNUMBER = 1
```

GOOD:

```lua
local my_number = 1
```


- Variables must have descriptive names

BAD: 

```lua
local obj = Find_First_Object("Star_Destroyer")	
```


GOOD:

```lua
local star_destroyer = Find_First_Object("Star_Destroyer")
```


- Use variables to avoid magic numbers

BAD:

```lua
local week_over = seconds_into_current_week > 40
```

GOOD:

```lua
local week_length = 40
local week_over = seconds_into_current_week > week_length
```


- If a variable represents a global constant its name should be written completely in capital letters

BAD:

```lua
week_length = 40
```


GOOD:

```lua
WEEK_LENGTH = 40
```


- Beware of upvalues (`local` variables defined outside the user’s scope). They cause save game crashes.

BAD:

```lua
local say_hello = "Hello World!"

function print_say_hello()
	DebugMessage(say_hello)
end

print_say_hello()
```

POSSIBLE SOLUTIONS:

- Global variable (discouraged, use only when upvalue is defined in root scope of the script)

```lua
Say_Hello = "Hello World"

function print_say_hello()
	DebugMessage(Say_Hello)
end

print_say_hello()
```

- Use table with method

```lua
local say_hello = {
	value = "Hello World",
	print = function(self)
		DebugMessage(self.value)
	end
}

say_hello:print()
```


## Functions
- Functions and function parameters must have descriptive names

BAD:

```lua
function helper(x, y)
	return x > y
end
```


GOOD:

```lua
function is_week_over(seconds_into_current_week, week_length)
	return seconds_into_current_week > week_length
end
```


- Annotate functions at least with type parameters and return type. For functions that perform complex tasks also add a description if the function name is not clear enough

```lua
---Returns true if the current week is over
---@param seconds_into_current_week number
---@param week_length number
---@return boolean
function is_week_over(seconds_into_current_week, week_length)
	return seconds_into_current_week > week_length
end
```

- Functions should not be long. If a function is more than 25 lines of code it should be split into multiple functions


## Conditional Statements
- Avoid nesting when possible. Use early returns instead

BAD:

```lua
---@param condition boolean
function a_function_that_needs_to_check_things(condition)
	if condition then
		local result = do_first_thing()
		if result then
			do_second_thing()
		end
	end
end
```


GOOD:

```lua
---@param condition boolean
function a_function_that_needs_to_check_things(condition)
	if not condition then
		return
	end

	local result = do_first_thing()
	
	if not result then
		return
	end

	do_second_thing()
end 
```


- Replace complex conditions with functions

BAD:

```lua
function do_something_for_complex_condition()
	if condition1 and condition2 or condition3 and condition4 or condition5 then
	end
end
```


GOOD:

```lua
---@return boolean
function conditions_fulfilled()
	return condition1 and condition2 or condition3 and condition4 or condition5
end

function do_something_for_complex_condition()
	if conditions_fulfilled() then
	end
end
```


## Tables
- Keep table value types consistent for array-like tables

BAD:

```lua
local my_table = { 1, "a_string", true }
```


GOOD:

```lua
local a_number_table = {1, 2, 3}
local a_string_table = { "a", "b", "c" }
```

- Table keys should follow the variable guidelines unless they represent string values returned from `Object.Get_Name()` function or a constant. In that case the keys should be in all capital letters

BAD:

```lua
local a_table_with_keys = {
	MY_KEY = 5,
	My_Other_Key = "hello",
	MyThirdKey = true
}
```


GOOD:

```lua
local a_table_with_keys = {
	my_key = 5,
	my_other_key = "hello",
	my_third_key = true
}
```


GOOD:

```lua
local a_table_with_keys = {
	EMPIRE = "the_empire"
}

local a_table_with_a_constant = {
	THE_CONSTANT = "always the same"
}
```

- Add type annotation when type can’t be inferred (for example empty tables)

GOOD:

```lua
---@type number[]
local array_like_table_with_numbers = {}

---@type table<string, string>
local a_table_with_string_keys_string_values = {}
```


## Loops
- Break or return from loops early if possible

BAD:

```lua
function get_index_of_a_thing(array_like_table, the_thing)
	local the_index
	for index, value in ipairs(array_like_table) do
		if value == the_thing then
			the_index = index
		end
	end

	return the_index
end
```


GOOD:

```lua
function get_index_of_a_thing(array_like_table, the_thing)
	for index, value in ipairs(array_like_table) do
		if value == the_thing then
			return index
		end
	end
end
```


- If loops become too long or complex or has a lot of nesting extract either entire loop or part of loop into another function

BAD:

```lua
function a_function_with_a_complex_loop(array_like)
	for index, value in ipairs(array_like) do
		if condition then
			local result = do_something_with(index, value)
			if result then
				do_something_else(result)
			end
		end
	end
end
```


GOOD:

```lua
function do_something_with_index_value(index, value)
	if not condition then
		return
	end
	
	local result = do_something_with(index, value)
	
	if not result then
		return
	end
	
	do_something_else(result)
end

function a_function_with_a_complex_loop(array_like)
	for index, value in ipairs(array_like) do
		do_something_with_index_value(index, value)
	end
end
```


- Add type annotation for loop parameters if they can’t be inferred

```lua
---@param key string
---@param value PlanetObject
for key, value in pairs(map_like) do
end
```

- Use `ipairs()` for array-like and `pairs()` for map-like tables

BAD:

```lua
local array_like = { 1, 2, 3 }
for index, value in pairs(array_like) do
end
```


GOOD:

```lua
local array_like = { 1, 2, 3 }
for index, value in ipairs(array_like) do
end
```


## Classes

- Class names are written in `PascalCase`

BAD:

```lua
my_class = class()
MYCLASS = class()
My_Class = class()
```


GOOD:

```lua
MyClass = class()
```


- Only one class should be defined per file
- At the end of the file the class should be returned

```lua
MyClass = class()
-- ...

return MyClass
```


- The constructor must a function called `new()`

```lua
MyClass = class()

function MyClass:new()
end
```

- Classes should have a `@class` annotation

```lua
---@class MyClass
MyClass = class()
```

- Attribute and method names must follow the table key guidelines
- If an attribute type can’t be inferred add type annotations
- Methods must have type annotations (see functions)
- Classes should be global variables due to EaW’s upvalue restrictions
