Tools allow Claude to access information from the outside world, extending its capabilities beyond what it learned during training. By default, Claude only knows information from its training data and can't access current events, real-time data, or external systems. Tool use solves this limitation by creating a structured way for Claude to request and receive fresh information.

## **The Problem Without Tools**

When users ask Claude for current information, it hits a wall. For example, if someone asks "What's the weather in San Francisco, California?" Claude has to respond with something like "I'm sorry, but I don't have access to up-to-date weather information."

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623642%2F06_-_001_-_Introducing_Tool_Use_05.1748623642311.png)

This creates a frustrating user experience when people need real-time data that Claude could theoretically help with if it just had access to current information.

## **How Tool Use Works**

Tool use follows a specific back-and-forth pattern between your application and Claude. Here's the complete flow:

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623643%2F06_-_001_-_Introducing_Tool_Use_07.1748623643055.png)

1. **Initial Request:** You send Claude a question along with instructions on how to get extra data from external sources
2. **Tool Request:** Claude analyzes the question and decides it needs additional information, then asks for specific details about what data it needs
3. **Data Retrieval:** Your server runs code to fetch the requested information from external APIs or databases
4. **Final Response:** You send the retrieved data back to Claude, which then generates a complete response using both the original question and the fresh data



## **Weather Example in Practice**

Let's see how this works with the weather question. The process becomes much more specific:

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623644%2F06_-_001_-_Introducing_Tool_Use_14.1748623643863.png)

When a user asks about current weather, you include instructions in your prompt about how to retrieve weather data. Claude recognizes it needs current information and requests weather data for the specific location. Your server then calls a weather API to get real-time conditions and sends that data back to Claude. Finally, Claude combines the fresh weather data with the user's question to provide an accurate, current response.

## **Key Benefits**

- **Real-time Information:** Access current data that wasn't available during Claude's training
- **External System Integration:** Connect Claude to databases, APIs, and other services
- **Dynamic Responses:** Provide answers based on the latest available information
- **Structured Interaction:** Claude knows exactly what information it needs and how to ask for it

Tool use transforms Claude from a static knowledge base into a dynamic assistant that can work with live data. This opens up possibilities for building applications that need current information, whether that's weather data, stock prices, database queries, or any other real-time information your users might need.

We're going to build a practical project that teaches Claude how to set reminders for future dates. This might sound simple at first, but it reveals several interesting challenges that we'll solve using custom tools.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623630%2F06_-_002_-_Project_Overview_00.1748623629884.png)

The goal is straightforward: we want to be able to tell Claude "Set a reminder for my doctor's appointment. It's a week from Thursday" and have Claude respond with "OK, I will remind you." But to make this work, we need to address some limitations in how Claude handles time and reminders.

## **Why This Is Challenging**

While Claude knows the current date, there are three specific problems we need to solve:

- **Limited time awareness:** Claude might know the current date, but not the exact time
- **Date calculation issues:** Claude doesn't always handle time-based addition well, especially when looking many days into the future
- **No reminder capability:** Claude doesn't know how to set a reminder - it has no built-in mechanism for this

Each of these limitations represents a gap between what Claude can do naturally and what we need for our reminder system. Tools are how we bridge these gaps.

## **Tools We Need**

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623630%2F06_-_002_-_Project_Overview_17.1748623630589.png)

We'll create three separate tools to handle each challenge:

- **Get the current date time:** Claude needs to know the current date and time precisely
- **Add duration to date time:** Claude isn't perfect with date time addition, so we'll give it a reliable tool for this
- **Set a reminder:** We need a way to actually set a reminder in the system

We'll implement these tools one at a time, starting with the simplest one. This approach lets us understand how tool calling works before building more complex functionality. By the end, Claude will be able to handle natural language requests like "remind me in a week" by combining these tools to calculate the exact time and set the reminder.

This project demonstrates a key principle of working with AI: when the model has limitations, we extend its capabilities through tools rather than trying to work around those limitations in our prompts.

After writing your tool function, the next step is creating a JSON schema that tells Claude what arguments your function expects and how to use it. This schema acts as documentation that Claude reads to understand when and how to call your tools.

## **Understanding JSON Schema**

JSON Schema isn't specific to AI or tool calling - it's a widely-used data validation specification that's been around for years. The AI community adopted it because it's a convenient way to describe function parameters and validate data.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623699%2F06_-_004_-_Tool_Schemas_01.1748623699583.png)

The complete tool specification has three main parts:

- **name** - A clear, descriptive name for your tool (like "get_weather")
- **description** - What the tool does, when to use it, and what it returns
- **input_schema** - The actual JSON schema describing the function's arguments

When working with Claude's tool functionality, you'll encounter a new type of response structure that's different from the simple text responses you've seen before. Instead of just getting back a single text block, Claude can now return multi-block messages that contain both text and tool usage information.

## **Making Tool-Enabled API Calls**

To enable Claude to use tools, you need to include a `tools` parameter in your API call. Here's how to structure the request:

```
messages = []
messages.append({
    "role": "user",
    "content": "What is the exact time, formatted as HH:MM:SS?"
})

response = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    tools=[get_current_datetime_schema],
)
```

The `tools` parameter takes a list of JSON schemas that describe the available functions Claude can call.

## **Understanding Multi-Block Messages**

When Claude decides to use a tool, it returns an assistant message with multiple blocks in the content list. This is a significant change from the simple text-only responses you've worked with before.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623695%2F06_-_005_-_Handling_Message_Blocks_07.1748623695372.png)

A multi-block message typically contains:

- **Text Block** - Human-readable text explaining what Claude is doing (like "I can help you find out the current time. Let me find that information for you")
- **ToolUse Block** - Instructions for your code about which tool to call and what parameters to use

The ToolUse block includes:

- An ID for tracking the tool call
- The name of the function to call (like "get_current_datetime")
- Input parameters formatted as a dictionary
- The type designation "tool_use"



## **Managing Conversation History with Multi-Block Messages**

Remember that Claude doesn't store conversation history - you need to manage it manually. When working with tool responses, you must preserve the entire content structure, including all blocks.

Here's how to properly append a multi-block assistant message to your conversation history:

```
messages.append({
    "role": "assistant",
    "content": response.content
})
```

This preserves both the text block and the tool use block, which is crucial for maintaining the conversation context when you make subsequent API calls.

## **The Complete Tool Usage Flow**

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623696%2F06_-_005_-_Handling_Message_Blocks_15.1748623696291.png)

The tool usage process follows this pattern:

1. Send user message with tool schema to Claude
2. Receive assistant message with text block and tool use block
3. Extract tool information and execute the actual function
4. Send tool result back to Claude along with complete conversation history
5. Receive final response from Claude

Each step requires careful handling of the message structure to ensure Claude has the full context it needs to provide accurate responses.

## **Updating Helper Functions**

If you've been using helper functions like `add_user_message()` and `add_assistant_message()`, you'll need to update them to handle multi-block content. The current versions likely only support single text blocks, but now they need to accommodate the more complex content structures that include tool use blocks.

This multi-block message handling is essential for building robust applications that can seamlessly integrate Claude's tool capabilities while maintaining proper conversation flow.

After Claude requests a tool call, you need to execute the function and send the results back. This completes the tool use workflow by providing Claude with the information it requested.

## **Running the Tool Function**

When Claude responds with a tool use block, you extract the input parameters and call your function. Here's how to access the tool parameters:

```
response.content[1].input
```

This gives you a dictionary of the arguments Claude wants to pass to your function. Since your function expects keyword arguments rather than a dictionary, you use Python's unpacking syntax:

```
get_current_datetime(**response.content[1].input)
```

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623700%2F06_-_006_-_Sending_Tool_Results_03.1748623700363.png)

## **Tool Result Block**

After running the tool function, you need to send the results back to Claude using a tool result block. This block goes inside a user message and tells Claude what happened when you executed the tool.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623701%2F06_-_006_-_Sending_Tool_Results_05.1748623701131.png)

The tool result block has several important properties:

- **tool_use_id** - Must match the id of the ToolUse block that this ToolResult corresponds to
- **content** - Output from running your tool, serialized as a string
- **is_error** - True if an error occurred



## **Handling Multiple Tool Calls**

Claude can request multiple tool calls in a single response. For example, if a user asks "What's 10 + 10 and what's 30 + 30?", Claude might respond with two separate ToolUse blocks.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623702%2F06_-_006_-_Sending_Tool_Results_07.1748623702036.png)

Each tool call gets a unique ID, and you must match these IDs when sending back results. This ensures Claude knows which result corresponds to which request, even if the results arrive in a different order.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623703%2F06_-_006_-_Sending_Tool_Results_08.1748623703510.png)

## **Building the Follow-up Request**

Your follow-up request to Claude must include the complete conversation history plus the new tool result. Here's the structure:

```
messages.append({
    "role": "user",
    "content": [{
        "type": "tool_result",
        "tool_use_id": response.content[1].id,
        "content": "15:04:22",
        "is_error": False
    }]
})
```

The complete message history now contains:

- Original user message
- Assistant message with tool use block
- User message with tool result block



## **Making the Final Request**

When sending the follow-up request, you must still include the tool schema even though you're not expecting Claude to make another tool call. Claude needs the schema to understand the tool references in your conversation history.

```
client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    tools=[get_current_datetime_schema]
)
```

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623704%2F06_-_006_-_Sending_Tool_Results_04.1748623704156.png)

Claude will then respond with a final message that incorporates the tool results into a natural response for the user. The tool use workflow is now complete - you've successfully enabled Claude to access real-time information through your custom function.

Building a conversation system with tools requires implementing a loop that keeps calling Claude until it stops requesting tool usage. When Claude no longer asks for tools, that signals it has a final response ready for the user.

## **Detecting Tool Requests**

The key to knowing whether Claude wants to use a tool lies in the `stop_reason` field of the response message. When Claude decides it needs to call a tool, this field gets set to `"tool_use"`. This gives us a clean way to check if we need to continue the conversation loop:

```
if response.stop_reason != "tool_use":
    break  # Claude is done, no more tools needed
```



## **The Conversation Loop**

The main conversation function follows a simple pattern:

```
def run_conversation(messages):
    while True:
        response = chat(messages, tools=[get_current_datetime_schema])
        add_assistant_message(messages, response)
        print(text_from_message(response))
        
        if response.stop_reason != "tool_use":
            break
            
        tool_results = run_tools(response)
        add_user_message(messages, tool_results)
    
    return messages
```

This loop continues until Claude provides a final answer without requesting any tools.

## **Handling Multiple Tool Calls**

Claude can request multiple tools in a single response. The message content contains a list of blocks, and we need to process each tool use block separately:

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623771%2F06_-_008_-_Implementing_Multiple_Turns_05.1748623771473.png)

The `run_tools` function handles this by filtering for tool use blocks and processing each one:

```
def run_tools(message):
    tool_requests = [
        block for block in message.content if block.type == "tool_use"
    ]
    tool_result_blocks = []
    
    for tool_request in tool_requests:
        # Process each tool request...
```



## **Tool Result Blocks**

Each tool use block must be answered with a corresponding tool result block. The connection between them is maintained through matching IDs:

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623772%2F06_-_008_-_Implementing_Multiple_Turns_10.1748623772471.png)

The tool result block structure includes:

```
tool_result_block = {
    "type": "tool_result",
    "tool_use_id": tool_request.id,
    "content": json.dumps(tool_output),
    "is_error": False
}
```



## **Error Handling**

Robust tool execution requires handling potential errors. When a tool fails, we still need to provide a result block to Claude:

```
try:
    tool_output = run_tool(tool_request.name, tool_request.input)
    tool_result_block = {
        "type": "tool_result",
        "tool_use_id": tool_request.id,
        "content": json.dumps(tool_output),
        "is_error": False
    }
except Exception as e:
    tool_result_block = {
        "type": "tool_result", 
        "tool_use_id": tool_request.id,
        "content": f"Error: {e}",
        "is_error": True
    }
```



## **Scalable Tool Routing**

To support multiple tools, create a routing function that maps tool names to their implementations:

```
def run_tool(tool_name, tool_input):
    if tool_name == "get_current_datetime":
        return get_current_datetime(**tool_input)
    elif tool_name == "another_tool":
        return another_tool(**tool_input)
    # Add more tools as needed
```

This approach makes it easy to add new tools without modifying the core conversation logic.

## **Complete Workflow**

The complete multi-turn conversation works like this:

- Send user message to Claude with available tools
- Claude responds with text and/or tool requests
- Execute all requested tools and create result blocks
- Send tool results back as a user message
- Repeat until Claude provides a final answer

This creates a seamless experience where Claude can use multiple tools across several turns to fully answer complex user requests. The conversation history maintains the complete context, allowing Claude to build upon previous tool results to provide comprehensive responses.

Adding multiple tools to your Claude implementation becomes straightforward once you have the core tool-handling infrastructure in place. This tutorial shows how to integrate additional tools by following a simple pattern.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623759%2F06_-_009_-_Using_Multiple_Tools_00.1748623759640.png)

## **The Tools We're Adding**

We need three main capabilities for our reminder system:

- **Get current date time** - Claude needs to know the current date and time
- **Add duration to date time** - Claude isn't perfect with date time addition
- **Set a reminder** - Need a way to set a reminder

The good news is that most of the implementation work is already done. The `add_duration_to_datetime` function and `set_reminder` function are provided, along with their corresponding schemas.

## **Adding Tools to the Conversation**

First, update the `run_conversation` function to include the new tool schemas in the tools list:

```
response = chat(messages, tools=[
    get_current_datetime_schema,
    add_duration_to_datetime_schema,
    set_reminder_schema
])
```

This tells Claude about all three available tools it can use during the conversation.

## **Updating the Tool Router**

Next, modify the `run_tool` function to handle the new tool calls. Add elif cases for each new tool:

```
def run_tool(tool_name, tool_input):
    if tool_name == "get_current_datetime":
        return get_current_datetime(**tool_input)
    elif tool_name == "add_duration_to_datetime":
        return add_duration_to_datetime(**tool_input)
    elif tool_name == "set_reminder":
        return set_reminder(**tool_input)
```

The pattern is simple: check the tool name, call the corresponding function with the provided input, and return the result.

## **Testing Multiple Tool Usage**

To test the system, try a request that requires multiple tools: "Set a reminder for my doctors appointment. Its 177 days after Jan 1st, 2050."

This request forces Claude to:

1. Calculate the date (using `add_duration_to_datetime`)
2. Set the reminder (using `set_reminder`)

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623760%2F06_-_009_-_Using_Multiple_Tools_15.1748623760531.png)

Claude handles this by first explaining what it needs to do, then making the appropriate tool calls in sequence. The conversation shows Claude calculating June 27, 2050 as the target date, then setting the reminder for that date.

## **Understanding the Message Flow**

When you examine the conversation history, you'll see the complete message structure:

- User message with the request
- Assistant message containing both text and tool use blocks
- Tool result messages
- Follow-up assistant messages

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623761%2F06_-_009_-_Using_Multiple_Tools_18.1748623761688.png)

This demonstrates how Claude can include multiple blocks in a single message - combining explanatory text with tool usage requests.

## **The Simple Pattern for Adding Tools**

Once you have the core tool infrastructure, adding new tools follows this pattern:

1. Create the tool function implementation
2. Define the tool schema
3. Add the schema to the tools list in `run_conversation`
4. Add a case for the tool in `run_tool`

This modular approach makes it easy to expand your AI assistant's capabilities without restructuring existing code. Each new tool integrates seamlessly with the existing conversation flow and tool-handling logic.

When you combine tool use with streaming in Claude, you get real-time updates as the AI generates tool arguments. This creates a more responsive user experience, but there are some important details to understand about how it works behind the scenes.

## **Basic Tool Streaming**

With streaming enabled, Claude sends back different types of events as it processes your request. You're already familiar with events like `ContentBlockDelta` for regular text generation. For tool use, you'll also need to handle a new event type called `InputJsonEvent`.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1752775508%2F06_-_011.1_-_Fine_Grained_Tool_Calling_01.1752775507859.png)

Each `InputJsonEvent` contains two key properties:

- **partial_json** - A chunk of JSON representing part of the tool arguments
- **snapshot** - The cumulative JSON built up from all chunks received so far

Here's how you handle these events in your streaming pipeline:

```
for chunk in stream:
    if chunk.type == "input_json":
        # Process the partial JSON chunk
        print(chunk.partial_json)
        # Or use the complete snapshot so far
        current_args = chunk.snapshot

```

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1752775508%2F06_-_011.1_-_Fine_Grained_Tool_Calling_02.1752775508676.png)

## **How JSON Validation Works**

Here's where things get interesting. The Anthropic API doesn't immediately send you every chunk as Claude generates it. Instead, it buffers chunks and validates them first.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1752775509%2F06_-_011.1_-_Fine_Grained_Tool_Calling_08.1752775509598.png)

The API waits for complete top-level key-value pairs before sending anything. For example, if your tool expects this structure:

```
{
  "abstract": "This paper presents a novel...",
  "meta": {
    "word_count": 847,
    "review": "This paper introduces QuanNet..."
  }
}
```

The API will:

1. Wait until the entire `abstract` value is complete
2. Validate that key-value pair against your schema
3. Send all the buffered chunks for `abstract` at once
4. Repeat the process for the `meta` object

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1752775510%2F06_-_011.1_-_Fine_Grained_Tool_Calling_10.1752775510417.png)

This validation process explains why you see delays followed by bursts of text, even with streaming enabled. The chunks are being held back until a complete, valid top-level key-value pair is ready.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1752775511%2F06_-_011.1_-_Fine_Grained_Tool_Calling_11.1752775511555.png)

## **Fine-Grained Tool Calling**

If you need faster, more granular streaming - perhaps to show users immediate updates or start processing partial results quickly - you can enable fine-grained tool calling.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1752775512%2F06_-_011.1_-_Fine_Grained_Tool_Calling_13.1752775512126.png)

Fine-grained tool calling does one main thing: it disables JSON validation on the API side. This means:

- You get chunks as soon as Claude generates them
- No buffering delays between top-level keys
- More traditional streaming behavior
- **Critical:** JSON validation is disabled - your code must handle invalid JSON

Enable it by adding `fine_grained=True` to your API call:

```
run_conversation(
    messages, 
    tools=[save_article_schema], 
    fine_grained=True
)
```

With fine-grained tool calling, you might receive a `word_count` value much earlier in the stream, without waiting for the entire `meta` object to be completed.

## **Handling Invalid JSON**

When using fine-grained tool calling, Claude might generate invalid JSON like `"word_count": undefined` instead of a proper number. Your application needs to handle these cases gracefully:

```
try:
    parsed_args = json.loads(chunk.snapshot)
except json.JSONDecodeError:
    # Handle invalid JSON appropriately
    print("Received invalid JSON, continuing...")

```

Without fine-grained tool calling, the API's validation would catch this error and potentially wrap problematic values in strings, which might not match your expected schema.

## **When to Use Fine-Grained Tool Calling**

Consider enabling fine-grained tool calling when:

- You need to show users real-time progress on tool argument generation
- You want to start processing partial tool results as quickly as possible
- The buffering delays negatively impact your user experience
- You're comfortable implementing robust JSON error handling

For most applications, the default behavior with validation is perfectly adequate. But when you need that extra responsiveness, fine-grained tool calling gives you the control to get chunks as fast as Claude can generate them.

**Important Note: Tool version strings can for all model versions can be found here:** [https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/text-editor-tool](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/text-editor-tool)

Claude comes with one built-in tool that you don't need to create from scratch: the text editor tool. This tool gives Claude the ability to work with files and directories just like you would in a standard text editor.

## **What the Text Editor Tool Can Do**

The text editor tool provides Claude with a comprehensive set of file manipulation capabilities:

- View file or directory contents
- View specific ranges of lines in a file
- Replace text in a file
- Create new files
- Insert text at specific lines in a file
- Undo recent edits to files

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623830%2F06_-_012_-_The_Text_Edit_Tool_00.1748623830120.png)

This dramatically expands Claude's abilities and essentially gives it the power to act as a software engineer right out of the gate.

## **Understanding the Implementation Requirements**

Here's where things get a bit confusing: while the tool schema is built into Claude, you still need to provide the actual implementation. Think of it this way - Claude knows how to ask for file operations, but you need to write the code that actually performs those operations.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623831%2F06_-_012_-_The_Text_Edit_Tool_04.1748623831083.png)

When you use other tools, you write both the JSON schema and the function implementation. With the text editor tool, Claude provides the schema knowledge, but you must write functions to handle Claude's requests to create files, read directories, replace text, and so on.

## **Schema Versions**

While the main schema is built into Claude, you do need to include a small schema stub when making requests. The exact schema depends on which Claude model you're using:

```
def get_text_edit_schema(model):
    if model.startswith("claude-3-7-sonnet"):
        return {
            "type": "text_editor_20250124",
            "name": "str_replace_editor",
        }
    elif model.startswith("claude-3-5-sonnet"):
        return {
            "type": "text_editor_20241022", 
            "name": "str_replace_editor",
        }
```

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623832%2F06_-_012_-_The_Text_Edit_Tool_12.1748623832064.png)

Claude sees this small schema and automatically expands it into the full text editor tool specification behind the scenes.

## **Practical Example**

Let's see the text editor tool in action. When you ask Claude to work with files, it will use the tool to read, modify, and create files as needed.

For example, if you ask Claude to "Open the ./[main.py](http://main.py) file and summarize its contents", Claude will:

1. Use the text editor tool to view the file
2. Read the contents
3. Provide you with a summary

You can take this further by asking Claude to modify files. For instance: "Open the ./[main.py](http://main.py) file and write out a function to calculate pi to the 5th digit. Then create a ./[test.py](http://test.py) file to test your implementation."

Claude will:

1. View the existing [main.py](http://main.py) file
2. Replace its contents with a new implementation including the pi calculation function
3. Create a new [test.py](http://test.py) file with appropriate unit tests



## **Why Use the Text Editor Tool?**

You might wonder why this tool exists when modern code editors already have AI assistants built in. The text editor tool becomes valuable in scenarios where:

- You're building applications that need to programmatically edit files
- You're working in environments without access to full-featured code editors
- You want to integrate file editing capabilities directly into your Claude-powered applications

Essentially, the text editor tool lets you replicate much of the functionality of a fancy AI-powered code editor within your own applications, giving you fine-grained control over how Claude interacts with your file system.



**Important note: Your organization must enable the Web Search tool in the settings console before using it. You can find this setting here:** [https://console.anthropic.com/settings/privacy](https://console.anthropic.com/settings/privacy)

Claude includes a built-in web search tool that lets it search the internet for current or specialized information to answer user questions. Unlike other tools where you need to provide the implementation, Claude handles the entire search process automatically - you just need to provide a simple schema to enable it.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623824%2F06_-_013_-_The_Web_Search_Tool_00.1748623823785.png)

## **Setting Up the Web Search Tool**

To use the web search tool, you create a schema object with these required fields:

```
web_search_schema = {
    "type": "web_search_20250305",
    "name": "web_search", 
    "max_uses": 5
}
```

The `max_uses` field limits how many searches Claude can perform. Claude might do follow-up searches based on initial results, so this prevents excessive API calls. A single search returns multiple results, but Claude may decide additional searches are needed.

## **How the Response Works**

When Claude uses the web search tool, the response contains several types of blocks:

- **Text blocks** - Claude's explanation of what it's doing
- **ServerToolUseBlock** - Shows the exact search query Claude used
- **WebSearchToolResultBlock** - Contains the search results
- **WebSearchResultBlock** - Individual search results with titles and URLs
- **Citation blocks** - Text that supports Claude's statements

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623825%2F06_-_013_-_The_Web_Search_Tool_07.1748623824808.png)

The response structure lets you see exactly what Claude searched for and which sources it found. Citations include the specific text Claude used to support its answers, along with the source URLs.

## **Restricting Search Domains**

You can limit searches to specific domains using the `allowed_domains` field. This is particularly useful when you want reliable, authoritative sources:

```
web_search_schema = {
    "type": "web_search_20250305",
    "name": "web_search",
    "max_uses": 5,
    "allowed_domains": ["nih.gov"]
}
```

For example, when asking about medical or exercise advice, restricting to domains like PubMed ([nih.gov](http://nih.gov)) ensures you get evidence-based information rather than random blog content.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623825%2F06_-_013_-_The_Web_Search_Tool_13.1748623825691.png)

## **Rendering Search Results**

The different block types in the response are designed for specific UI rendering:

- Render text blocks as regular content
- Display web search results as a list of sources at the top
- Show citations inline with the text, including the source domain, page title, URL, and quoted text

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623826%2F06_-_013_-_The_Web_Search_Tool_17.1748623826456.png)

This structure helps users understand how Claude arrived at its answers and provides transparency about the sources being used. The citation format makes it clear which specific information came from which sources, building trust in the AI's responses.

## **Practical Usage**

The web search tool works best for:

- Current events and recent developments
- Specialized information not in Claude's training data
- Fact-checking and finding authoritative sources
- Research tasks requiring up-to-date information

Simply include the schema in your tools array when making API calls, and Claude will automatically decide when a web search would help answer the user's question.