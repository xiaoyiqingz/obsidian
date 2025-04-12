
At its core, MCP follows a client-server architecture where a host application can connect to multiple servers:

```mermaid
flowchart LR
    subgraph Computer["Your Computer"]
        Client["Host with MCP Client\n(Claude, IDEs, Tools)"]
        
        ServerA["MCP Server A"]
        DataA["Local\nData Source A"]
        
        ServerB["MCP Server B"]
        DataB["Local\nData Source B"]
        
        ServerC["MCP Server C"]
    end
    
    subgraph Internet
        ServiceC["Remote\nService C"]
    end
    
    Client <--"MCP Protocol"--> ServerA
    Client <--"MCP Protocol"--> ServerB
    Client <--"MCP Protocol"--> ServerC
    
    ServerA <--> DataA
    ServerB <--> DataB
    ServerC <--"Web APIs"--> ServiceC
    
    style Computer fill:#ffffdd,stroke:#999
    style Internet fill:#ffffdd,stroke:#999
    style Client fill:#e6e6fa,stroke:#666
    style ServerA fill:#e6e6fa,stroke:#666
    style ServerB fill:#e6e6fa,stroke:#666
    style ServerC fill:#e6e6fa,stroke:#666
    style DataA fill:#e6e6fa,stroke:#666,stroke-dasharray: 5 5
    style DataB fill:#e6e6fa,stroke:#666,stroke-dasharray: 5 5
    style ServiceC fill:#e6e6fa,stroke:#666,stroke-dasharray: 5 5

```
- **MCP Hosts**: Programs like Claude Desktop, IDEs, or AI tools that want to access data through MCP
- **MCP Clients**: Protocol clients that maintain 1:1 connections with servers
- **MCP Servers**: Lightweight programs that each expose specific capabilities through the standardized Model Context Protocol
- **Local Data Sources**: Your computer’s files, databases, and services that MCP servers can securely access
- **Remote Services**: External systems available over the internet (e.g., through APIs) that MCP servers can connect to


When you submit a query:

1. The client gets the list of available tools from the server
2. Your query is sent to Claude along with tool descriptions
3. Claude decides which tools (if any) to use
4. The client executes any requested tool calls through the server
5. Results are sent back to Claude
6. Claude provides a natural language response
7. The response is displayed to you

one server can provide multi tools,  like  github mcp server can provide:
* create_or_update_file
* search_repositories
* create_repository
* get_file_contents
* push_files
* create_issue
* create_pull_request
* fork_repository
* create_branch
* list_commits