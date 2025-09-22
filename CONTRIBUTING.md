## Contributing to Spec Kit
## Rules Table

| Rule ID | Rule Summary                                                           |
| ------- | ---------------------------------------------------------------------- |
| R1      | Use functional error handling with Result monad                       |
| R2      | Use Maybe monad for optional values instead of None                   |
| R3      | Apply Single Responsibility Principle - one purpose per method        |
| R4      | Use structural pattern matching for type-based dispatch               |
| R5      | Implement polymorphism via abstract methods over isinstance checks    |
| R6      | Use @overload for multiple function signatures                        |
| R7      | Maximum 15 lines per function/method                                  |
| R8      | Maximum 5 branch points (McCabe complexity) per unit                  |
| R9      | Never copy/paste code blocks - extract shared functionality           |
| R10     | Maximum 2 parameters per function/method                              |
| R11     | Separate concerns into distinct modules                               |
| R12     | Organize code by features, not technical layers                       |
| R13     | Use strict type annotations for all functions and classes             |
| R14     | Comprehensive input validation and sanity checks                      |
| R15     | Use loguru for structured logging at appropriate levels               |
| R16     | Encapsulate data and methods within classes                           |
| R17     | Remove all dead code and commented-out sections                       |
| R18     | Use component-based architecture for reusable functionality           |
| R19     | Implement layered architecture with clear dependencies                |
| R20     | Group related parameters into objects                                 |

---

## R1: Use functional error handling with Result monad

```python
# ❌ BAD - Traditional exception handling
def evaluate_xpath(self, xpath_expr: str, context_node: XPathNode):
    try:
        parser = XPathParser(self.tokenizer)
        parser.set_input(xpath_expr)
        ast = parser.parse()
        return self.evaluate(ast, context)
    except Exception as e:
        logger.error(f"Error: {e}")
        raise

# ✅ GOOD - Functional error handling with Result
from returns.result import Result, safe

@safe
def evaluate_xpath(
    self,
    xpath_expr: str,
    context_node: XPathNode,
) -> Result[XPathValue, Exception]:
    parser = XPathParser(self.tokenizer)
    parser.set_input(xpath_expr)
    ast = parser.parse()
    return self.evaluate(ast, context)

# Usage
result = evaluator.evaluate_xpath("//book/title", context_node)
if result.is_success:
    value = result.unwrap()
else:
    error = result.failure()
```

## R2: Use Maybe monad for optional values instead of None

```python
# ❌ BAD - Returning None
def get_function(self, name: str):
    return self.functions.get(name)  # Returns None if not found

# Usage requires None checks
func = get_function("last")
if func is not None:
    result = func.execute()

# ✅ GOOD - Using Maybe monad
from returns.maybe import Maybe, maybe

@maybe
def get_function(self, name: str) -> Maybe[XPathFunction]:
    return self.functions.get(name)

# Usage with explicit handling
maybe_func = get_function("last")
if maybe_func.is_some:
    func = maybe_func.unwrap()
    result = func.execute()
```

## R3: Apply Single Responsibility Principle - one purpose per method

```python
# ❌ BAD - Multiple responsibilities
def evaluate(self, node: XPathASTNode, context: XPathContext):
    if isinstance(node, XPathFunctionCall):
        # 20 lines of function call logic
        ...
    elif isinstance(node, XPathLiteral):
        # 15 lines of literal logic
        ...
    elif isinstance(node, XPathOperator):
        # 25 lines of operator logic
        ...

# ✅ GOOD - Single responsibility per method
def evaluate(self, node: XPathASTNode, context: XPathContext) -> XPathValue:
    if isinstance(node, XPathFunctionCall):
        return self._evaluate_function_call(node, context)
    elif isinstance(node, XPathLiteral):
        return self._evaluate_literal(node, context)
    elif isinstance(node, XPathOperator):
        return self._evaluate_operator(node, context)

def _evaluate_function_call(self, node: XPathFunctionCall, context: XPathContext):
    # Focused implementation
    ...
```

## R4: Use structural pattern matching for type-based dispatch

```python
# ❌ BAD - Multiple if-elif checks
def evaluate(self, node: XPathASTNode, context: XPathContext):
    if isinstance(node, XPathFunctionCall):
        return self._handle_function(node, context)
    elif isinstance(node, XPathLiteral):
        return self._handle_literal(node, context)
    else:
        raise ValueError(f"Unknown node: {type(node)}")

# ✅ GOOD - Pattern matching (Python 3.10+)
def evaluate(self, node: XPathASTNode, context: XPathContext) -> XPathValue:
    match node:
        case XPathFunctionCall(name=name, arguments=args):
            return self._handle_function(name, args, context)
        case XPathLiteral(value=value):
            return XPathValue(value)
        case XPathOperator(operator=op, operands=operands):
            return self._handle_operator(op, operands, context)
        case _:
            raise ValueError(f"Unsupported node: {type(node).__name__}")
```

## R5: Implement polymorphism via abstract methods over isinstance checks

```python
# ❌ BAD - External type checking
class Evaluator:
    def evaluate(self, node: XPathASTNode, context: XPathContext):
        if isinstance(node, XPathFunctionCall):
            # Function logic
        elif isinstance(node, XPathLiteral):
            # Literal logic

# ✅ GOOD - Polymorphic dispatch
from abc import ABC, abstractmethod

class XPathASTNode(ABC):
    @abstractmethod
    def evaluate(self, context: XPathContext) -> XPathValue:
        pass

class XPathFunctionCall(XPathASTNode):
    def evaluate(self, context: XPathContext) -> XPathValue:
        # Self-contained evaluation logic
        args = [arg.evaluate(context) for arg in self.arguments]
        return self.function.execute(context, args)

class XPathLiteral(XPathASTNode):
    def evaluate(self, context: XPathContext) -> XPathValue:
        return XPathValue(self.value)
```

## R6: Use @overload for multiple function signatures

```python
# ❌ BAD - Single signature with runtime type checking
def process(self, data):
    if isinstance(data, str):
        return self._process_string(data)
    elif isinstance(data, list):
        return self._process_list(data)

# ✅ GOOD - Overloaded signatures
from typing import overload, List

@overload
def process(self, data: str) -> str: ...

@overload
def process(self, data: List[str]) -> List[str]: ...

def process(self, data):
    if isinstance(data, str):
        return self._process_string(data)
    elif isinstance(data, list):
        return self._process_list(data)
```

## R7: Maximum 15 lines per function/method

```python
# ❌ BAD - Long function (violates maintainability guideline)
def process_document(self, content: str) -> Document:
    if not content:
        raise ValueError("Empty content")
    lines = content.strip().split('\n')
    sections = []
    current_section = None
    for line in lines:
        if line.startswith('##'):
            if current_section:
                sections.append(current_section)
            current_section = Section(line[2:].strip())
        elif current_section:
            current_section.add_line(line)
    if current_section:
        sections.append(current_section)
    # ... 10 more lines

# ✅ GOOD - Focused functions under 15 lines (follows "Write Short Units of Code" principle)
def process_document(self, content: str) -> Document:
    validated = self._validate_content(content)
    lines = self._extract_lines(validated)
    sections = self._parse_sections(lines)
    return Document(sections)

def _validate_content(self, content: str) -> str:
    if not content:
        raise ValueError("Empty content")
    return content.strip()

def _parse_sections(self, lines: List[str]) -> List[Section]:
    sections = []
    current = None
    for line in lines:
        current = self._process_line(line, current, sections)
    if current:
        sections.append(current)
    return sections
```

## R8: Maximum 5 branch points (McCabe complexity) per unit

```python
# ❌ BAD - High complexity (7 branch points, violates "Write Simple Units of Code")
def validate_user(self, user):
    if user is None:
        return False
    if not user.email:
        return False
    if '@' not in user.email:
        return False
    if user.age < 0:
        return False
    if user.age > 150:
        return False
    if not user.name:
        return False
    return True

# ✅ GOOD - Low complexity (max 4 branch points, follows simplicity principle)
def validate_user(self, user) -> bool:
    return (
        self._is_valid_object(user) and
        self._is_valid_email(user.email) and
        self._is_valid_age(user.age) and
        self._is_valid_name(user.name)
    )

def _is_valid_email(self, email: str) -> bool:
    return bool(email and '@' in email)

def _is_valid_age(self, age: int) -> bool:
    return 0 <= age <= 150
```

## R9: Never copy/paste code blocks - extract shared functionality

```python
# ❌ BAD - Duplicated code
def process_user(self, data):
    if not data:
        logger.error("No data provided")
        raise ValueError("No data")
    validated = self.validator.validate(data)
    result = self.transformer.transform(validated)
    return result

def process_order(self, data):
    if not data:
        logger.error("No data provided")
        raise ValueError("No data")
    validated = self.validator.validate(data)
    result = self.transformer.transform(validated)
    return result

# ✅ GOOD - Extracted common functionality
def _process_entity(self, data):
    if not data:
        logger.error("No data provided")
        raise ValueError("No data")
    validated = self.validator.validate(data)
    return self.transformer.transform(validated)

def process_user(self, data):
    return self._process_entity(data)

def process_order(self, data):
    return self._process_entity(data)
```

## R10: Maximum 2 parameters per function/method

```python
# ❌ BAD - Too many parameters
def create_user(self, name, email, age, address, phone):
    # Implementation
    pass

# ✅ GOOD - Group related parameters
@dataclass
class UserData:
    name: str
    email: str
    age: int
    address: str
    phone: str

def create_user(self, user_data: UserData):
    # Implementation using user_data.name, user_data.email, etc.
    pass
```

## R11: Separate concerns into distinct modules

```python
# ❌ BAD - Mixed concerns in one module
# user_manager.py
class UserManager:
    def create_user(self): ...
    def validate_email(self): ...
    def send_notification(self): ...
    def log_activity(self): ...

# ✅ GOOD - Separated concerns
# user/service.py
class UserService:
    def create_user(self): ...

# user/validator.py
class UserValidator:
    def validate_email(self): ...

# user/notifier.py
class UserNotifier:
    def send_notification(self): ...

# user/logger.py
class ActivityLogger:
    def log_activity(self): ...
```

## R12: Organize code by features, not technical layers

```python
# ❌ BAD - Organization by technical layers
project/
├── models/
│   ├── user.py
│   ├── order.py
├── services/
│   ├── user_service.py
│   ├── order_service.py
├── controllers/
│   ├── user_controller.py
│   ├── order_controller.py

# ✅ GOOD - Organization by features
project/
├── user/
│   ├── __init__.py
│   ├── models.py
│   ├── service.py
│   ├── controller.py
│   ├── validator.py
├── order/
│   ├── __init__.py
│   ├── models.py
│   ├── service.py
│   ├── controller.py
│   ├── validator.py
```

## R13: Use strict type annotations for all functions and classes

```python
# ❌ BAD - Missing or incomplete type annotations
def process(data):
    result = transform(data)
    return result

class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

# ✅ GOOD - Complete type annotations
from typing import Optional, Dict, List, Final

def process(data: Dict[str, Any]) -> ProcessResult:
    result: TransformResult = transform(data)
    return ProcessResult(result)

class User:
    name: Final[str]
    email: Final[str]
    
    def __init__(self, name: str, email: str) -> None:
        self.name = name
        self.email = email
    
    def get_info(self) -> Dict[str, str]:
        return {"name": self.name, "email": self.email}
```

## R14: Comprehensive input validation and sanity checks

```python
# ❌ BAD - No validation
def calculate_discount(price, percentage):
    return price * (1 - percentage / 100)

# ✅ GOOD - Comprehensive validation
def calculate_discount(price: float, percentage: float) -> float:
    # Sanity checks - fail fast
    if not isinstance(price, (int, float)):
        raise TypeError(f"Price must be numeric, got {type(price)}")
    if not isinstance(percentage, (int, float)):
        raise TypeError(f"Percentage must be numeric, got {type(percentage)}")
    if price < 0:
        raise ValueError(f"Price cannot be negative: {price}")
    if percentage < 0 or percentage > 100:
        raise ValueError(f"Percentage must be 0-100: {percentage}")
    
    return price * (1 - percentage / 100)
```

## R15: Use loguru for structured logging at appropriate levels

```python
# ❌ BAD - Print statements or basic logging
def process_request(request):
    print(f"Processing: {request}")
    try:
        result = handle(request)
        print(f"Success: {result}")
    except Exception as e:
        print(f"Error: {e}")

# ✅ GOOD - Structured loguru logging
from loguru import logger

def process_request(request: Request) -> Response:
    logger.info(f"Processing request: {request.id}")
    logger.trace(f"Request details: {request}")
    
    try:
        result = handle(request)
        logger.debug(f"Request {request.id} handled successfully")
        return result
    except ValidationError as e:
        logger.warning(f"Validation failed for {request.id}: {e}")
        raise
    except Exception as e:
        logger.error(f"Unexpected error processing {request.id}: {e}")
        raise
```

## R16: Encapsulate data and methods within classes

```python
# ❌ BAD - Exposed data structure
user_data = {
    'name': 'John',
    'email': 'john@example.com',
    'age': 30
}

def update_email(data, new_email):
    data['email'] = new_email

# ✅ GOOD - Encapsulated class
class User:
    def __init__(self, name: str, email: str, age: int):
        self._name = name
        self._email = email
        self._age = age
    
    @property
    def email(self) -> str:
        return self._email
    
    def update_email(self, new_email: str) -> None:
        if '@' not in new_email:
            raise ValueError("Invalid email format")
        self._email = new_email
```

## R17: Remove all dead code and commented-out sections

```python
# ❌ BAD - Dead code and comments
def process(data):
    # Old implementation
    # result = old_transform(data)
    # if result:
    #     return result
    
    result = new_transform(data)
    
    # TODO: Remove after testing
    # debug_value = calculate_debug(result)
    
    unused_var = "not used anywhere"
    
    return result

# ✅ GOOD - Clean, active code only
def process(data: Data) -> Result:
    return new_transform(data)
```

## R18: Use component-based architecture for reusable functionality

```python
# ❌ BAD - Monolithic structure
class ApplicationService:
    def handle_user_creation(self): ...
    def handle_order_processing(self): ...
    def handle_payment(self): ...
    def handle_notification(self): ...

# ✅ GOOD - Component-based architecture
# components/user/
class UserComponent:
    def __init__(self, validator: Validator, repository: Repository):
        self.validator = validator
        self.repository = repository
    
    def create_user(self, data: UserData) -> User:
        validated = self.validator.validate(data)
        return self.repository.save(validated)

# components/order/
class OrderComponent:
    def __init__(self, validator: Validator, repository: Repository):
        self.validator = validator
        self.repository = repository
    
    def process_order(self, data: OrderData) -> Order:
        validated = self.validator.validate(data)
        return self.repository.save(validated)
```

## R19: Implement layered architecture with clear dependencies

```python
# ❌ BAD - Mixed layers
def handle_request(request):
    # Direct database access
    conn = database.connect()
    data = conn.query("SELECT * FROM users")
    
    # Business logic mixed with data access
    if len(data) > 100:
        data = filter_premium_users(data)
    
    # Response formatting
    return json.dumps(data)

# ✅ GOOD - Clear layers
# Presentation Layer
class UserController:
    def __init__(self, service: UserService):
        self.service = service
    
    def handle_request(self, request: Request) -> Response:
        users = self.service.get_users(request.filters)
        return Response(users)

# Business Layer
class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository
    
    def get_users(self, filters: Filters) -> List[User]:
        users = self.repository.find_all()
        if len(users) > 100:
            users = self._filter_premium(users)
        return users

# Data Layer
class UserRepository:
    def find_all(self) -> List[User]:
        # Database interaction only
        return self.db.query("SELECT * FROM users")
```

## R20: Group related parameters into objects

```python
# ❌ BAD - Many related parameters
def send_email(to_address, from_address, subject, body, cc_list, bcc_list, attachments):
    # Implementation
    pass

# ✅ GOOD - Grouped into data class
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class EmailMessage:
    to_address: str
    from_address: str
    subject: str
    body: str
    cc_list: Optional[List[str]] = None
    bcc_list: Optional[List[str]] = None
    attachments: Optional[List[str]] = None

def send_email(message: EmailMessage) -> None:
    # Access via message.to_address, message.subject, etc.
    pass
```

## Prerequisites for running and testing code

These are one time installations required to be able to test your changes locally as part of the pull request (PR) submission process.

1. Install [Python 3.11+](https://www.python.org/downloads/)
1. Install [uv](https://docs.astral.sh/uv/) for package management
1. Install [Git](https://git-scm.com/downloads)
1. Have an [AI coding agent available](README.md#-supported-ai-agents)

## Submitting a pull request

>[!NOTE]
>If your pull request introduces a large change that materially impacts the work of the CLI or the rest of the repository (e.g., you're introducing new templates, arguments, or otherwise major changes), make sure that it was **discussed and agreed upon** by the project maintainers. Pull requests with large changes that did not have a prior conversation and agreement will be closed.

1. Fork and clone the repository
1. Configure and install the dependencies: `uv sync`
1. Make sure the CLI works on your machine: `uv run specify --help`
1. Create a new branch: `git checkout -b my-branch-name`
1. Make your change, add tests, and make sure everything still works
1. Test the CLI functionality with a sample project if relevant
1. Push to your fork and submit a pull request
1. Wait for your pull request to be reviewed and merged.

Here are a few things you can do that will increase the likelihood of your pull request being accepted:

- Follow the project's coding conventions.
- Write tests for new functionality.
- Update documentation (`README.md`, `spec-driven.md`) if your changes affect user-facing features.
- Keep your change as focused as possible. If there are multiple changes you would like to make that are not dependent upon each other, consider submitting them as separate pull requests.
- Write a [good commit message](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html).
- Test your changes with the Spec-Driven Development workflow to ensure compatibility.

## Development workflow

When working on spec-kit:

1. Test changes with the `specify` CLI commands (`/specify`, `/plan`, `/tasks`) in your coding agent of choice
2. Verify templates are working correctly in `templates/` directory
3. Test script functionality in the `scripts/` directory
4. Ensure memory files (`memory/constitution.md`) are updated if major process changes are made

## AI contributions in Spec Kit

We welcome and encourage the use of AI tools to help improve Spec Kit! Many valuable contributions have been enhanced with AI assistance for code generation, issue detection, and feature definition.

### What we're looking for

When submitting AI-assisted contributions, please ensure they include:

- **Human understanding and testing** - You've personally tested the changes and understand what they do
- **Clear rationale** - You can explain why the change is needed and how it fits within Spec Kit's goals  
- **Concrete evidence** - Include test cases, scenarios, or examples that demonstrate the improvement
- **Your own analysis** - Share your thoughts on the end-to-end developer experience

### What we'll close

We reserve the right to close contributions that appear to be:

- Untested changes submitted without verification
- Generic suggestions that don't address specific Spec Kit needs
- Bulk submissions that show no human review or understanding

### Guidelines for success

The key is demonstrating that you understand and have validated your proposed changes. If a maintainer can easily tell that a contribution was generated entirely by AI without human input or testing, it likely needs more work before submission.

Contributors who consistently submit low-effort AI-generated changes may be restricted from further contributions at the maintainers' discretion.

## Resources

- [Spec-Driven Development Methodology](./spec-driven.md)
- [How to Contribute to Open Source](https://opensource.guide/how-to-contribute/)
- [Using Pull Requests](https://help.github.com/articles/about-pull-requests/)
- [GitHub Help](https://help.github.com)
