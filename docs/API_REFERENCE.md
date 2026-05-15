# API Reference

## Overview

This comprehensive API reference covers all public interfaces, classes, methods, and data structures available in VEXIS-Maestro.

## Core Components

### FivePhaseEngine

**Location**: `src/ai_agent/core_processing/five_phase_engine.py`

The main orchestration engine that coordinates the 5-phase pipeline execution process.

#### Class Definition

```python
class FivePhaseEngine:
    """5-Phase Pipeline Execution Engine for CLI AI Agent System.
    
    Implements the complete 5-phase architecture:
    - Phase 1: Command Suggestion
    - Phase 2: Command Extraction
    - Phase 3: Command Execution
    - Phase 4: Log Evaluation
    - Phase 5: Summary Generation
    """
    
    def __init__(self, provider: str = None, model: str = None, config: Optional[Dict[str, Any]] = None, 
                 telegram_bot: Optional[TelegramBotManager] = None):
        """Initialize the 5-phase engine.
        
        Args:
            provider: AI provider to use
            model: Model to use
            config: Configuration dictionary containing engine settings
            telegram_bot: Optional Telegram bot manager
        """
```

#### Methods

##### `execute_instruction`

```python
def execute_instruction(
    self, 
    user_prompt: str,
    conversation_history: Optional[ConversationHistory] = None,
    telegram_mode: bool = False,
    telegram_user_id: Optional[int] = None,
    cancel_event: Optional[threading.Event] = None
) -> PipelineContext:
    """Execute user instruction through the 5-phase pipeline.
    
    Args:
        user_prompt: Natural language instruction from user
        conversation_history: Conversation history for Telegram mode
        telegram_mode: Whether running in Telegram mode
        telegram_user_id: Telegram user ID for sending messages
        cancel_event: Optional cancellation event
        
    Returns:
        PipelineContext: Complete execution results
        
    Raises:
        ValidationError: If instruction is invalid
        ExecutionError: If execution fails
    """
```

#### Usage Example

```python
from ai_agent.core_processing.five_phase_engine import FivePhaseEngine

# Initialize engine
engine = FivePhaseEngine(provider="ollama", model="gemma3:4b")

# Process instruction
context = engine.execute_instruction("list files in current directory")

if context.current_phase == PipelinePhase.COMPLETED:
    print(f"Task completed successfully")
    print(f"Summary: {context.final_summary}")
else:
    print(f"Error: {context.error}")
```

### ModelRunner

**Location**: `src/ai_agent/external_integration/model_runner.py`

Unified AI provider abstraction supporting 13+ providers: Ollama (local), Google Gemini, OpenAI, Anthropic, xAI, Meta, Mistral AI, Microsoft Azure, Amazon Bedrock, Cohere, DeepSeek, Groq, and Together AI.

#### Class Definition

```python
class ModelRunner:
    """Unified AI model runner supporting multiple providers."""
    
    def __init__(
        self,
        provider: str = None,
        model: str = None,
        config: Optional[Dict[str, Any]] = None,
        auto_install_sdks: bool = False
    ):
        """Initialize model runner.
        
        Args:
            provider: AI provider to use
            model: Model to use
            config: Optional configuration dictionary
            auto_install_sdks: Whether to auto-install missing provider SDKs
        """
```

#### Methods

##### `run_model`

```python
def run_model(
    self,
    request: ModelRequest
) -> ModelResponse:
    """Run AI model for CLI Architecture with retry on validation failure.
    
    Args:
        request: ModelRequest containing task_type, prompt, and parameters
        
    Returns:
        ModelResponse: Model's response with metadata
        
    Raises:
        ValidationError: If request validation fails
        ProviderError: If provider fails
    """
```

##### `install_missing_sdks`

```python
def install_missing_sdks(
    self,
    providers: Optional[List[str]] = None,
    interactive: bool = True
) -> Dict[str, bool]:
    """Install missing SDKs for specified providers.
    
    Args:
        providers: List of provider names to install SDKs for
        interactive: Whether to prompt user for confirmation
        
    Returns:
        Dict[str, bool]: Mapping of provider to installation success status
    """
```

##### `show_sdk_status`

```python
def show_sdk_status(self, providers: Optional[List[str]] = None):
    """Show SDK installation status for providers.
    
    Args:
        providers: List of provider names to check status for
    """
```

#### Usage Example

```python
from ai_agent.external_integration.model_runner import ModelRunner, ModelRequest, TaskType

# Initialize runner
runner = ModelRunner(provider="ollama", model="gemma3:4b")

# Create request
request = ModelRequest(
    task_type=TaskType.PHASE1_COMMAND_SUGGESTION,
    prompt="Generate a command to list files",
    max_tokens=5000,
    temperature=1.0
)

# Run model
response = runner.run_model(request)

if response.success:
    print(f"Response: {response.content}")
    print(f"Model: {response.model}")
    print(f"Tokens used: {response.tokens_used}")
```

### CommandParser

**Location**: `src/ai_agent/core_processing/command_parser.py`

Converts natural language instructions into executable CLI commands.

#### Class Definition

```python
class CommandParser:
    """Parser for converting natural language to CLI commands."""
    
    def __init__(self, config: Dict[str, Any]):
        """Initialize command parser.
        
        Args:
            config: Configuration dictionary
        """
```

#### Methods

##### `parse_instruction`

```python
def parse_instruction(
    self,
    instruction: str,
    context: Optional[Dict[str, Any]] = None
) -> ParseResult:
    """Parse natural language instruction into commands.
    
    Args:
        instruction: Natural language instruction
        context: Optional context information
        
    Returns:
        ParseResult: Parsed commands and metadata
        
    Raises:
        ParseError: If parsing fails
        ValidationError: If instruction is invalid
    """
```

##### `validate_command`

```python
def validate_command(self, command: Command) -> ValidationResult:
    """Validate a command for safety and correctness.
    
    Args:
        command: Command to validate
        
    Returns:
        ValidationResult: Validation result with details
    """
```

##### `sanitize_command`

```python
def sanitize_command(self, command: str) -> str:
    """Sanitize command string for safe execution.
    
    Args:
        command: Raw command string
        
    Returns:
        str: Sanitized command string
    """
```

#### Usage Example

```python
from ai_agent.core_processing.command_parser import CommandParser

# Initialize parser
parser = CommandParser(config)

# Parse instruction
result = parser.parse_instruction("create a file named test.txt")

if result.success:
    for command in result.commands:
        print(f"Command: {command.executable}")
        print(f"Args: {command.arguments}")
        print(f"Safe: {command.is_safe}")
```

## Data Structures

### PipelineContext

**Location**: `src/ai_agent/core_processing/five_phase_engine.py`

```python
@dataclass
class PipelineContext:
    """Context for tracking 5-phase pipeline execution."""
    
    user_prompt: str
    phase1_output: Optional[str] = None
    input_summary: Optional[str] = None
    extracted_commands: Optional[str] = None
    terminal_log: str = ""
    phase4_output: Optional[str] = None
    final_summary: Optional[str] = None
    current_phase: PipelinePhase = PipelinePhase.PHASE1_COMMAND_SUGGESTION
    iteration_count: int = 0
    max_iterations: int = 500
    start_time: float = field(default_factory=time.time)
    end_time: Optional[float] = None
    error: Optional[str] = None
    metadata: Dict[str, Any] = field(default_factory=dict)
    conversation_history: Optional[ConversationHistory] = None
    telegram_mode: bool = False
    telegram_user_id: Optional[int] = None
    cancel_event: Optional[threading.Event] = None
    cancelled: bool = False
```

### PipelinePhase

**Location**: `src/ai_agent/core_processing/five_phase_engine.py`

```python
class PipelinePhase(Enum):
    """5-Phase Pipeline phases."""
    PHASE1_COMMAND_SUGGESTION = "phase1_command_suggestion"
    PHASE2_COMMAND_EXTRACTION = "phase2_command_extraction"
    PHASE3_COMMAND_EXECUTION = "phase3_command_execution"
    PHASE4_LOG_EVALUATION = "phase4_log_evaluation"
    PHASE5_SUMMARY_GENERATION = "phase5_summary_generation"
    COMPLETED = "completed"
    FAILED = "failed"
```

### TaskType

**Location**: `src/ai_agent/external_integration/model_runner.py`

```python
class TaskType(Enum):
    """Task types for 5-Phase CLI Architecture."""
    PHASE1_COMMAND_SUGGESTION = "phase1_command_suggestion"
    INPUT_SUMMARIZATION = "input_summarization"
    PHASE2_COMMAND_EXTRACTION = "phase2_command_extraction"
    PHASE4_LOG_EVALUATION = "phase4_log_evaluation"
    PHASE5_SUMMARY_GENERATION = "phase5_summary_generation"
```

### ModelRequest

**Location**: `src/ai_agent/external_integration/model_runner.py`

```python
@dataclass
class ModelRequest:
    """Model request structure."""
    
    task_type: TaskType
    prompt: str
    image_data: Optional[bytes] = None
    image_format: str = "PNG"
    context: Optional[Dict[str, Any]] = None
    parameters: Optional[Dict[str, Any]] = None
    max_tokens: int = 5000
    temperature: float = 1.0
    timeout: int = 30
```

### ModelResponse

**Location**: `src/ai_agent/external_integration/model_runner.py`

```python
@dataclass
class ModelResponse:
    """Model response structure."""
    
    success: bool
    content: str
    task_type: TaskType
    model: str
    provider: str
    tokens_used: Optional[int] = None
    cost: Optional[float] = None
    latency: Optional[float] = None
    error: Optional[str] = None
    metadata: Optional[Dict[str, Any]] = None
```

### ValidationError

**Location**: `src/ai_agent/utils/exceptions.py`

```python
class ValidationError(Exception):
    """Raised when input validation fails."""
    
    def __init__(
        self, 
        message: str, 
        field: Optional[str] = None,
        value: Optional[Any] = None
    ):
        """Initialize validation error.
        
        Args:
            message: Error message
            field: Field that failed validation
            value: Value that failed validation
        """
        super().__init__(message)
        self.field = field
        self.value = value
```

## Utilities

### Configuration

**Location**: `src/ai_agent/utils/config.py`

#### `load_config`

```python
def load_config(config_path: str = "config.yaml") -> Dict[str, Any]:
    """Load configuration from file.
    
    Args:
        config_path: Path to configuration file
        
    Returns:
        Dict[str, Any]: Loaded configuration
        
    Raises:
        ConfigError: If configuration loading fails
    """
```

#### `validate_config`

```python
def validate_config(config: Dict[str, Any]) -> ValidationResult:
    """Validate configuration dictionary.
    
    Args:
        config: Configuration to validate
        
    Returns:
        ValidationResult: Validation result
    """
```

### Logging

**Location**: `src/ai_agent/utils/logger.py`

#### `get_logger`

```python
def get_logger(name: str) -> logging.Logger:
    """Get configured logger instance.
    
    Args:
        name: Logger name
        
    Returns:
        logging.Logger: Configured logger
    """
```

#### `setup_logging`

```python
def setup_logging(config: Dict[str, Any]) -> None:
    """Set up logging configuration.
    
    Args:
        config: Logging configuration
    """
```

### Error Handler

**Location**: `src/ai_agent/utils/ollama_error_handler.py`

#### `handle_ollama_error`

```python
def handle_ollama_error(
    error_message: str,
    context: Optional[Dict[str, Any]] = None,
    display_to_user: bool = True
) -> Tuple[Optional[ErrorInfo], bool]:
    """Handle Ollama-related errors with user guidance.
    
    Args:
        error_message: Error message to handle
        context: Additional context information
        display_to_user: Whether to display guidance to user
        
    Returns:
        Tuple[Optional[ErrorInfo], bool]: Error info and whether to retry
    """
```

## User Interface

### Yellow Selection System

**Location**: `src/ai_agent/utils/yellow_selection/main.py`

#### `get_yellow_menu`

```python
def get_yellow_menu(
    title: str,
    description: str,
    use_fallback: bool = False
) -> CleanInteractiveMenu:
    """Get a yellow-highlighted interactive menu.
    
    Args:
        title: Menu title
        description: Menu description
        use_fallback: Use fallback menu implementation
        
    Returns:
        CleanInteractiveMenu: Menu instance
    """
```

#### `get_yellow_selector`

```python
def get_yellow_selector(use_fallback: bool = False) -> CleanHierarchicalSelector:
    """Get the yellow hierarchical selector.
    
    Returns:
        CleanHierarchicalSelector: Selector instance
    """
```

### Main Application

**Location**: `src/ai_agent/user_interface/five_phase_app.py`

#### `main`

```python
def main():
    """Main entry point for 5-Phase Pipeline AI Agent."""
```

## Integration Examples

### Basic Usage

```python
from ai_agent.core_processing.five_phase_engine import FivePhaseEngine

# Initialize engine
engine = FivePhaseEngine(provider="ollama", model="gemma3:4b")

# Process instruction
context = engine.execute_instruction(
    "list files in current directory"
)

if context.current_phase == PipelinePhase.COMPLETED:
    print("Success!")
    print(f"Summary: {context.final_summary}")
else:
    print(f"Error: {context.error}")
```

### Custom Provider

```python
from ai_agent.external_integration.model_runner import ModelRunner, ModelRequest, TaskType

# Use the multi-provider system directly
runner = ModelRunner(provider="custom", model="custom-model")

request = ModelRequest(
    task_type=TaskType.PHASE1_COMMAND_SUGGESTION,
    prompt="your instruction"
)

response = runner.run_model(request)
```

### Error Handling

```python
from ai_agent.utils.ollama_error_handler import handle_ollama_error
from ai_agent.utils.exceptions import ValidationError

def safe_instruction_processing(instruction: str):
    """Process instruction with comprehensive error handling."""
    
    try:
        # Validate instruction
        if not instruction.strip():
            raise ValidationError("Instruction cannot be empty")
        
        # Process with engine
        engine = FivePhaseEngine(provider="ollama", model="gemma3:4b")
        context = engine.execute_instruction(instruction)
        
        return context
        
    except ValidationError as e:
        print(f"Validation Error: {e}")
        return None
        
    except Exception as e:
        # Handle with enhanced error handler
        error_info, should_retry = handle_ollama_error(
            str(e),
            context={"instruction": instruction}
        )
        
        if should_retry:
            print("Retrying with alternative provider...")
            return safe_instruction_processing(instruction)
        else:
            print(f"Error: {error_info.message if error_info else str(e)}")
            return None
```

### Configuration Management

```python
from ai_agent.utils.config import load_config, validate_config

def setup_environment(config_path: str = "config.yaml"):
    """Set up environment with configuration validation."""
    
    try:
        # Load configuration
        config = load_config(config_path)
        
        # Validate configuration
        validation_result = validate_config(config)
        
        if not validation_result.is_valid:
            print("Configuration validation failed:")
            for error in validation_result.errors:
                print(f"  - {error}")
            return None
        
        # Set up logging
        from ai_agent.utils.logger import setup_logging
        setup_logging(config.get("logging", {}))
        
        return config
        
    except Exception as e:
        print(f"Configuration setup failed: {e}")
        return None
```

## Testing API

### Test Utilities

**Location**: `tests/utils/`

#### `MockModelRunner`

```python
class MockModelRunner:
    """Mock model runner for testing."""
    
    def __init__(self, responses: Dict[str, str]):
        self.responses = responses
    
    async def generate_response(
        self, 
        task_type: TaskType,
        prompt: str,
        **kwargs
    ) -> ModelResponse:
        """Generate mock response."""
        
        response_text = self.responses.get(
            prompt, 
            "Mock response for: " + prompt
        )
        
        return ModelResponse(
            success=True,
            content=response_text,
            task_type=task_type,
            model="mock-model",
            provider="mock",
            tokens_used=len(response_text.split()),
            latency=0.1
        )
```

#### `TestFixtures`

```python
@pytest.fixture
def sample_config():
    """Sample configuration for testing."""
    return {
        "api": {
            "preferred_provider": "ollama",
            "local_endpoint": "http://localhost:11434"
        },
        "execution": {
            "safety_mode": True,
            "dry_run": False
        }
    }

@pytest.fixture
def mock_ollama_response():
    """Mock Ollama API response."""
    return {
        "response": "ls -la",
        "done": True,
        "model": "gemini-3-flash-preview"
    }
```

## Migration Guide

### Version Compatibility

This API reference covers VEXIS-Maestro version 2.1.0.

#### Breaking Changes from 0.x

- `TwoPhaseEngine` replaced with `FivePhaseEngine` using 5-phase architecture
- `ModelRunner` now uses `run_model()` with `ModelRequest` instead of `generate_response()`
- TaskType enum updated to use phase-based task types
- Configuration format uses YAML with enhanced provider support

#### Migration Steps

1. Update imports:
   ```python
   # Old
   from ai_agent.engine import TwoPhaseEngine
   
   # New
   from ai_agent.core_processing.five_phase_engine import FivePhaseEngine
   ```

2. Update method calls:
   ```python
   # Old
   result = await engine.process_instruction("test")
   
   # New
   context = engine.execute_instruction("test")
   ```

3. Update configuration:
   ```python
   # Old JSON format
   config = {"preferred_provider": "ollama"}
   
   # New YAML format
   config = load_config("config.yaml")
   ```

This API reference provides comprehensive documentation for all public interfaces in VEXIS-Maestro, enabling developers to integrate, extend, and test the system effectively.
