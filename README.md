# Weaviate MCP Server

A [Model Context Protocol (MCP)](https://github.com/modelcontextprotocol/specification) server that provides tools for interacting with [Weaviate](https://weaviate.io/), a vector database. This server enables AI assistants and other MCP clients to insert objects and perform hybrid search queries on Weaviate collections.

## What is MCP?

The Model Context Protocol (MCP) is an open standard that enables AI assistants to securely connect to external data sources and tools. This Weaviate MCP server exposes Weaviate's capabilities through standardized MCP tools that can be used by any MCP-compatible AI assistant.

## Features

- **Object Insertion**: Insert objects with properties into Weaviate collections
- **Hybrid Search**: Perform semantic and keyword search queries across your data
- **Auto-schema**: Leverage Weaviate's automatic schema generation
- **Collection Management**: Work with multiple Weaviate collections
- **MCP Compliance**: Full compatibility with MCP specification

## Prerequisites

- **Go 1.23.1 or later**: Required for building the server
- **Weaviate Instance**: A running Weaviate database
  - Can be local (Docker) or cloud-hosted
  - Must be accessible at the configured endpoint
  - Default: `http://localhost:8080`

### Setting up Weaviate

The easiest way to run Weaviate locally is with Docker:

```bash
docker run -d \
  --name weaviate \
  -p 8080:8080 \
  -e QUERY_DEFAULTS_LIMIT=25 \
  -e AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED=true \
  -e PERSISTENCE_DATA_PATH='/var/lib/weaviate' \
  -e DEFAULT_VECTORIZER_MODULE='none' \
  -e ENABLE_MODULES='' \
  -e CLUSTER_HOSTNAME='node1' \
  semitechnologies/weaviate:latest
```

## Installation

### Building from Source

1. **Clone the repository**:
   ```bash
   git clone https://github.com/thedotmack/mcp-server-weaviate.git
   cd mcp-server-weaviate
   ```

2. **Build the server**:
   ```bash
   make build
   ```

   This creates the `mcp-server` binary in the `client/` directory.

3. **Verify the build**:
   ```bash
   ls -la client/mcp-server
   ```

### Dependencies

The server uses the following key dependencies:
- `github.com/mark3labs/mcp-go` - MCP protocol implementation
- `github.com/weaviate/weaviate-go-client/v4` - Weaviate Go client
- `github.com/weaviate/weaviate` - Weaviate entities and models

## Configuration

Currently, the server uses hardcoded configuration values. Future versions will support environment variables.

### Default Settings

- **Weaviate Host**: `localhost:8080`
- **Weaviate Scheme**: `http`
- **Default Collection**: `DefaultCollection`
- **Transport**: STDIO (standard input/output)

### Planned Environment Variables

The following environment variables are planned for future releases:
- `WEAVIATE_HOST` - Weaviate server host
- `WEAVIATE_SCHEME` - HTTP scheme (http/https)
- `WEAVIATE_DEFAULT_COLLECTION` - Default collection name

## Usage

### Starting the Server

The MCP server communicates via STDIO and is typically invoked by MCP clients:

```bash
./client/mcp-server
```

### Testing with the Included Client

A test client is provided to verify functionality:

```bash
make run-client
```

This will:
1. Start the MCP server
2. Insert a test object (Valencia, Spain location data)
3. Query for the inserted data
4. Display the results

## Tools

The server provides two MCP tools:

### weaviate-insert-one

Insert an object into a Weaviate collection.

**Parameters:**
- `collection` (string, optional): Target collection name. Uses default collection if not specified.
- `properties` (object, required): Object properties to insert.

**Example Request:**
```json
{
  "name": "weaviate-insert-one",
  "arguments": {
    "collection": "WorldMap",
    "properties": {
      "continent": "Europe",
      "country": "Spain",
      "city": "Valencia"
    }
  }
}
```

**Response:**
Returns the UUID of the inserted object as text:
```
"550e8400-e29b-41d4-a716-446655440000"
```

### weaviate-query

Perform hybrid search queries on Weaviate collections.

**Parameters:**
- `query` (string, required): Search query text.
- `targetProperties` (array, required): List of properties to return in results.
- `collection` (string, optional): Target collection name. Uses default collection if not specified.

**Example Request:**
```json
{
  "name": "weaviate-query",
  "arguments": {
    "collection": "WorldMap",
    "query": "What country is Valencia in?",
    "targetProperties": ["continent", "country", "city"]
  }
}
```

**Response:**
Returns JSON-formatted search results:
```json
{
  "data": {
    "Get": {
      "WorldMap": [
        {
          "continent": "Europe",
          "country": "Spain",
          "city": "Valencia"
        }
      ]
    }
  }
}
```

## Integration Examples

### Using with Claude Desktop

Add to your Claude Desktop configuration:

```json
{
  "mcpServers": {
    "weaviate": {
      "command": "/path/to/mcp-server-weaviate/client/mcp-server"
    }
  }
}
```

### Using with Other MCP Clients

The server works with any MCP-compatible client. Ensure your Weaviate instance is running and accessible before starting the server.

## Development

### Project Structure

```
.
├── main.go           # Entry point
├── mcp.go           # MCP server implementation and tool handlers
├── weaviate.go      # Weaviate client and database operations
├── client/
│   ├── client.go    # Test client implementation
│   └── mcp-server   # Built server binary
├── Makefile         # Build and run commands
├── go.mod           # Go module definition
└── README.md        # This file
```

### Building and Testing

1. **Build the server**:
   ```bash
   make build
   ```

2. **Run tests with the client**:
   ```bash
   make run-client
   ```

3. **Manual testing**:
   ```bash
   # Start Weaviate (if not already running)
   docker run -d -p 8080:8080 semitechnologies/weaviate:latest
   
   # Run the server
   ./client/mcp-server
   ```

### Code Overview

- **MCPServer**: Main server struct that handles MCP protocol and tool routing
- **WeaviateConnection**: Manages connection to Weaviate database
- **Tool Handlers**: Implement the insert and query operations
- **Batch Operations**: Uses Weaviate's batch API for efficient insertions

## Error Handling

The server includes comprehensive error handling:
- Validates required parameters
- Handles Weaviate connection errors
- Returns appropriate MCP error responses
- Includes detailed error messages for debugging

## Limitations

- No authentication support (uses anonymous access)
- Hardcoded configuration (environment variables planned)
- STDIO transport only (SSE transport planned)
- Limited query customization options

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with the included client
5. Submit a pull request

## License

This project is open source. Please refer to the repository for license information.
