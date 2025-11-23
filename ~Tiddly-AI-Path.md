# TiddlyWiki AI Orchestration: Implementation Tasks

This document provides a structured list of tasks to implement the AI orchestration control plane described in `~Tiddly-AI.md`. Tasks are organized into phases, from foundational infrastructure to advanced features and UX.

---

## Phase 0: Prerequisites & Setup

### 0.1 Environment & Dependencies
- [ ] Review and understand the TiddlyWiki5 codebase structure
- [ ] Set up development environment for TiddlyWiki plugins
- [ ] Install Node.js dependencies for server-side operations
- [ ] Create a test wiki instance for development
- [ ] Set up version control and branching strategy for AI features

### 0.2 Architecture Documentation
- [ ] Review `~Tiddly-AI.md` meta-model thoroughly
- [ ] Create architecture decision records (ADRs) for key design choices
- [ ] Document field naming conventions and namespaces
- [ ] Define API contracts between orchestrator and providers
- [ ] Create data flow diagrams for key operations

### 0.3 Security & Privacy Planning
- [ ] Design consent mechanism for external API calls
- [ ] Define privacy filter for sensitive content
- [ ] Plan API key management (storage, rotation, access control)
- [ ] Document data retention policies
- [ ] Create security review checklist

---

## Phase 1: Foundation Layer

### 1.1 Core Tiddler Types & Schema

#### 1.1.1 Define base tag types
- [ ] Create `AIPrompt` tag definition and documentation
- [ ] Create `AIQuestion` tag definition and documentation
- [ ] Create `AIAnswer` tag definition and documentation
- [ ] Create `AIRun` tag definition and documentation
- [ ] Create `AIBatch` tag definition and documentation
- [ ] Create `AIPipeline` tag definition and documentation
- [ ] Create `AIAgent` tag definition and documentation
- [ ] Create `AITool` tag definition and documentation
- [ ] Create `AIDataset` tag definition and documentation
- [ ] Create `AIEvalSpec` and `AIEvalRun` tag definitions
- [ ] Create `AIProfile` tag definition and documentation
- [ ] Create `AIQueue` tag definition and documentation
- [ ] Create `AIConversation` and `AIMessage` tag definitions

#### 1.1.2 Field namespace implementation
- [ ] Document all `ai:*` field conventions
- [ ] Create field validation utilities
- [ ] Implement field type checking (JSON, timestamps, enums)
- [ ] Create field serialization/deserialization helpers
- [ ] Add field migration utilities for schema evolution

#### 1.1.3 System tiddler structure
- [ ] Create `$:/ai/` directory structure
- [ ] Define `$:/ai/config/` for global settings
- [ ] Define `$:/ai/profile/` for model profiles
- [ ] Define `$:/ai/prompt/` for system prompts
- [ ] Define `$:/ai/queue/` for queue configurations
- [ ] Define `$:/ai/usage/` for usage tracking
- [ ] Define `$:/ai/policy/` for governance policies

### 1.2 Basic Utilities & Helpers

#### 1.2.1 Hashing & idempotency
- [ ] Implement content hashing function (for `ai:lastHash`)
- [ ] Implement request normalization for `ai:hash` on runs
- [ ] Create hash comparison utilities
- [ ] Add cache lookup by hash
- [ ] Implement idempotent run detection

#### 1.2.2 Timestamp & scheduling utilities
- [ ] Create ISO timestamp generation utilities
- [ ] Implement timestamp parsing and comparison
- [ ] Add duration calculation helpers
- [ ] Create scheduling/cron utilities for background jobs
- [ ] Implement lease expiration checking

#### 1.2.3 JSON field utilities
- [ ] Create JSON field getter/setter with validation
- [ ] Implement JSON schema validation for fields
- [ ] Add JSON merge/patch utilities
- [ ] Create JSON path query helpers
- [ ] Add error handling for malformed JSON fields

### 1.3 Provider Abstraction Layer

#### 1.3.1 Base provider interface
- [ ] Define provider interface contract (request/response)
- [ ] Create abstract base provider class
- [ ] Implement provider registration system
- [ ] Add provider capability detection
- [ ] Create provider selection logic based on profiles

#### 1.3.2 OpenAI provider implementation
- [ ] Implement OpenAI API client
- [ ] Add chat completions endpoint support
- [ ] Add embeddings endpoint support
- [ ] Implement streaming response handling
- [ ] Add error handling and retry logic
- [ ] Implement token counting and cost calculation
- [ ] Add rate limiting and quota management

#### 1.3.3 Anthropic provider implementation
- [ ] Implement Anthropic Claude API client
- [ ] Add message endpoint support
- [ ] Implement tool use (function calling) support
- [ ] Add streaming support
- [ ] Implement error handling and retries

#### 1.3.4 Local model provider implementation
- [ ] Design local model provider interface
- [ ] Add support for Ollama
- [ ] Add support for llama.cpp server
- [ ] Add support for LocalAI
- [ ] Implement model auto-detection
- [ ] Add offline capability checks

#### 1.3.5 Provider testing & mocking
- [ ] Create mock provider for testing
- [ ] Implement provider test suite
- [ ] Add integration tests for each provider
- [ ] Create provider benchmark utilities
- [ ] Add provider health check endpoints

---

## Phase 2: Core Orchestration Engine

### 2.1 Queue System

#### 2.1.1 Queue implementation
- [ ] Implement in-memory queue data structure
- [ ] Add persistent queue storage (in wiki)
- [ ] Create queue management API (enqueue, dequeue, peek)
- [ ] Implement priority queue support
- [ ] Add FIFO/LIFO queue modes
- [ ] Implement queue filtering by tags/fields

#### 2.1.2 Queue worker & scheduler
- [ ] Create queue worker/consumer loop
- [ ] Implement concurrency control (max concurrent runs)
- [ ] Add rate limiting (requests per minute)
- [ ] Implement burst handling
- [ ] Add backpressure mechanisms
- [ ] Create worker status monitoring

#### 2.1.3 Run lifecycle management
- [ ] Implement run state machine (`queued → running → succeeded/failed`)
- [ ] Add run creation from questions
- [ ] Implement run execution logic
- [ ] Add retry logic with exponential backoff
- [ ] Implement cancellation support
- [ ] Add run timeout handling
- [ ] Create run result persistence

#### 2.1.4 Lease & lock management
- [ ] Implement run leasing for distributed workers
- [ ] Add lease renewal logic
- [ ] Implement lease expiration and reclaim
- [ ] Add dead letter queue for stuck runs
- [ ] Create lease debugging tools

### 2.2 Profile & Budget Management

#### 2.2.1 Profile system
- [ ] Implement profile loading from `$:/ai/profile/*` tiddlers
- [ ] Add profile inheritance/composition
- [ ] Create profile validation
- [ ] Implement profile selection logic
- [ ] Add profile override mechanism (question → standard → profile)

#### 2.2.2 Budget tracking
- [ ] Implement cost calculation per run
- [ ] Add budget ledger updates
- [ ] Create monthly usage rollup
- [ ] Implement budget cap enforcement (per-run, per-day, per-month)
- [ ] Add budget alerts and warnings
- [ ] Create budget reporting utilities

#### 2.2.3 Token & latency tracking
- [ ] Implement token counting for requests/responses
- [ ] Add latency measurement per run
- [ ] Create token usage aggregation
- [ ] Implement latency percentile tracking
- [ ] Add throughput metrics (runs/minute)

### 2.3 Prompt & Question Management

#### 2.3.1 Prompt rendering
- [ ] Implement variable interpolation for prompts (`<<variable name>>` or `${name}`)
- [ ] Add template rendering engine
- [ ] Create prompt validation (check for undefined variables)
- [ ] Implement prompt composition (merge system + user prompts)
- [ ] Add prompt versioning support
- [ ] Create prompt testing utilities

#### 2.3.2 Standard question implementation
- [ ] Implement standard question loading
- [ ] Add question instance creation from standard
- [ ] Create variable binding (topic → prompt variables)
- [ ] Implement profile override logic
- [ ] Add cost cap enforcement
- [ ] Create question status tracking

#### 2.3.3 Question-to-run conversion
- [ ] Implement question → run conversion logic
- [ ] Add batch creation for sampling questions
- [ ] Create run request payload generation
- [ ] Implement provider selection for runs
- [ ] Add run priority assignment

---

## Phase 3: Sampling & Meta-Analysis

### 3.1 Sampling Pattern

#### 3.1.1 Batch sampling
- [ ] Implement batch creation from question
- [ ] Add N-sample run generation
- [ ] Create batch progress tracking
- [ ] Implement batch completion detection
- [ ] Add batch cancellation support
- [ ] Create batch result aggregation

#### 3.1.2 Answer collection
- [ ] Implement answer tiddler creation from run results
- [ ] Add answer linking (to question, batch, run)
- [ ] Create answer deduplication logic
- [ ] Implement answer storage optimization
- [ ] Add answer versioning

### 3.2 Meta-Question System

#### 3.2.1 Meta-question implementation
- [ ] Create meta-question loader
- [ ] Implement answer collection via filters
- [ ] Add meta-prompt rendering (include multiple answers)
- [ ] Create meta-run execution
- [ ] Implement meta-result parsing

#### 3.2.2 Ranking meta-operation
- [ ] Implement "rank answers" meta-question
- [ ] Add ranking prompt template
- [ ] Create ranking result parser (scores, ranks)
- [ ] Implement answer score updates
- [ ] Add ranking visualization utilities

#### 3.2.3 Disagreement detection
- [ ] Implement "find disagreements" meta-question
- [ ] Add disagreement detection prompt
- [ ] Create disagreement highlighting
- [ ] Implement disagreement resolution workflow
- [ ] Add disagreement reporting

#### 3.2.4 Final synthesis
- [ ] Implement "synthesize final answer" meta-question
- [ ] Add synthesis prompt with top-ranked answers
- [ ] Create canonical answer tiddler
- [ ] Implement answer linking (question → final answer)
- [ ] Add synthesis quality metrics

---

## Phase 4: Pipeline & Workflow Engine

### 4.1 Pipeline Definition & Loading

#### 4.1.1 Pipeline schema
- [ ] Define pipeline step schema (id, type, spec, inputs, outputs, dependencies)
- [ ] Implement pipeline validation
- [ ] Add pipeline dependency graph construction
- [ ] Create cycle detection in pipeline DAG
- [ ] Implement pipeline versioning

#### 4.1.2 Pipeline loader
- [ ] Create pipeline tiddler parser
- [ ] Implement step loader with dependency resolution
- [ ] Add pipeline caching
- [ ] Create pipeline hot-reload support

### 4.2 Pipeline Execution

#### 4.2.1 Pipeline runner
- [ ] Implement pipeline execution engine
- [ ] Add topological sort for step ordering
- [ ] Create step execution with input/output mapping
- [ ] Implement step retry and error handling
- [ ] Add pipeline pause/resume support
- [ ] Create pipeline cancellation

#### 4.2.2 Step types
- [ ] Implement "sample" step type (spawn N runs)
- [ ] Implement "meta" step type (meta-question over answers)
- [ ] Implement "transform" step type (tiddler transformation)
- [ ] Implement "import" step type (external data import)
- [ ] Implement "filter" step type (tiddler filtering/selection)
- [ ] Implement "compose" step type (nested pipeline)

#### 4.2.3 Pipeline state management
- [ ] Create pipeline run tracking
- [ ] Implement step status tracking
- [ ] Add pipeline progress calculation
- [ ] Create pipeline result aggregation
- [ ] Implement pipeline lineage tracking (step → outputs)

---

## Phase 5: Agent System

### 5.1 Agent Definition & Loading

#### 5.1.1 Agent schema
- [ ] Define agent tiddler schema (persona, tools, triggers, mode)
- [ ] Implement agent validation
- [ ] Create agent loading and caching
- [ ] Add agent versioning

#### 5.1.2 Tool system
- [ ] Define tool tiddler schema (name, description, input/output schemas, implementation)
- [ ] Implement tool registration
- [ ] Create tool validation (schema checks)
- [ ] Add tool discovery (list available tools)

### 5.2 Agent Execution

#### 5.2.1 Agent conversation management
- [ ] Implement conversation creation
- [ ] Add message appending
- [ ] Create conversation context management
- [ ] Implement conversation history truncation/summarization
- [ ] Add conversation persistence

#### 5.2.2 Tool calling
- [ ] Implement tool call detection from LLM response
- [ ] Create tool dispatcher
- [ ] Add tool execution sandbox
- [ ] Implement tool result formatting
- [ ] Create tool call retry logic

#### 5.2.3 Agent modes
- [ ] Implement "manual" mode (user-triggered)
- [ ] Implement "on-save" mode (trigger on tiddler save)
- [ ] Implement "scheduled" mode (cron-based)
- [ ] Implement "continuous" mode (always-on monitoring)
- [ ] Add mode selection and configuration

### 5.3 Built-in Tools

#### 5.3.1 Wiki tools
- [ ] Implement `SearchWiki` tool (filter-based search)
- [ ] Implement `GetTiddler` tool (retrieve tiddler content)
- [ ] Implement `ListTiddlers` tool (list by filter)
- [ ] Implement `CreateTiddler` tool (create draft/proposal)
- [ ] Implement `UpdateTiddler` tool (propose changes)
- [ ] Implement `DeleteTiddler` tool (propose deletion)

#### 5.3.2 External tools
- [ ] Implement `HttpGet` tool (fetch external content)
- [ ] Implement `HttpPost` tool (post to webhooks)
- [ ] Implement `WikipediaSearch` tool
- [ ] Implement `WikipediaGet` tool (fetch article)
- [ ] Add tool for web scraping (with consent)

---

## Phase 6: Evaluation & Metrics

### 6.1 Dataset Management

#### 6.1.1 Dataset schema & loading
- [ ] Define dataset tiddler schema
- [ ] Implement dataset item loading
- [ ] Add dataset validation
- [ ] Create dataset versioning
- [ ] Implement dataset import/export

### 6.2 Evaluation Execution

#### 6.2.1 Eval spec implementation
- [ ] Create eval spec loader
- [ ] Implement eval run creation
- [ ] Add per-item evaluation execution
- [ ] Create eval result aggregation
- [ ] Implement eval comparison utilities

#### 6.2.2 Metrics
- [ ] Implement exact match metric
- [ ] Implement BLEU metric (if relevant)
- [ ] Implement ROUGE metric (if relevant)
- [ ] Implement LLM-as-judge metric
- [ ] Add custom metric plugin support
- [ ] Create metric visualization

#### 6.2.3 Leaderboard
- [ ] Create eval run comparison UI
- [ ] Implement prompt/pipeline leaderboard
- [ ] Add metric filtering and sorting
- [ ] Create historical tracking (metric trends over time)

---

## Phase 7: Import & Sync

### 7.1 Wikipedia Integration

#### 7.1.1 Wikipedia import
- [ ] Implement Wikipedia API client
- [ ] Add article search
- [ ] Add article fetch (full text)
- [ ] Create article-to-tiddler mapping
- [ ] Implement chunking for long articles
- [ ] Add image/media handling

#### 7.1.2 Import pipeline
- [ ] Create "import from Wikipedia" pipeline template
- [ ] Add topic-to-Wikipedia mapping
- [ ] Implement conflict detection (wiki vs. Wikipedia)
- [ ] Create merge/update workflow
- [ ] Add change tracking (what's new/changed)

### 7.2 Other Sources

#### 7.2.1 Generic source adapters
- [ ] Define source spec schema (URL, API, auth, mapping)
- [ ] Implement generic HTTP source
- [ ] Add RSS/Atom feed source
- [ ] Create API source with auth
- [ ] Implement file import source

#### 7.2.2 Sync & refresh
- [ ] Implement periodic refresh (cron-based)
- [ ] Add incremental updates (only fetch changes)
- [ ] Create sync status tracking
- [ ] Implement conflict resolution UI
- [ ] Add rollback/revert support

---

## Phase 8: UX & Visualization

### 8.1 Control Panel Integration

#### 8.1.1 AI control panel
- [ ] Create `$:/ControlPanel` AI tab
- [ ] Add provider configuration UI
- [ ] Add profile management UI
- [ ] Add queue configuration UI
- [ ] Add budget tracking dashboard
- [ ] Add policy/consent management

### 8.2 Topic-Centric Views

#### 8.2.1 Topic tiddler enhancements
- [ ] Add "AI Answer" section to topic view
- [ ] Create "Ask more" panel for topics
- [ ] Add "Sources" section (Wikipedia, docs)
- [ ] Implement sub-question suggestions
- [ ] Add disagreement/issue highlighting

#### 8.2.2 Question UI
- [ ] Create question creation UI
- [ ] Add question status display
- [ ] Implement sampling progress bar
- [ ] Add answer list view
- [ ] Create final answer display

### 8.3 Run & Queue Dashboards

#### 8.3.1 Queue dashboard
- [ ] Create queue status overview (runs by queue/status)
- [ ] Add run list with filtering
- [ ] Implement run detail view
- [ ] Add run cancellation buttons
- [ ] Create run requeue functionality

#### 8.3.2 Run inspector
- [ ] Create detailed run view (request, response, timing, cost)
- [ ] Add request/response JSON viewers
- [ ] Implement run comparison view
- [ ] Add run lineage view (question → run → answer)
- [ ] Create run debugging tools

### 8.4 Pipeline Builder UI

#### 8.4.1 Pipeline editor
- [ ] Create visual pipeline editor (drag-and-drop steps)
- [ ] Add step configuration forms
- [ ] Implement dependency edge drawing
- [ ] Add pipeline validation display
- [ ] Create pipeline testing/dry-run mode

#### 8.4.2 Pipeline execution view
- [ ] Create pipeline run status display
- [ ] Add step-by-step progress view
- [ ] Implement step result preview
- [ ] Add pipeline re-run support
- [ ] Create step-level retry

### 8.5 Agent Playground

#### 8.5.1 Agent conversation UI
- [ ] Create chat interface for agents
- [ ] Add message input/output display
- [ ] Implement tool call visualization
- [ ] Add agent configuration panel
- [ ] Create conversation history view

#### 8.5.2 Agent management
- [ ] Create agent list view
- [ ] Add agent creation wizard
- [ ] Implement tool assignment UI
- [ ] Add agent testing/playground
- [ ] Create agent performance metrics

### 8.6 Evaluation Dashboard

#### 8.6.1 Eval runs view
- [ ] Create eval run list
- [ ] Add per-item results table
- [ ] Implement metric charts
- [ ] Add eval comparison view
- [ ] Create leaderboard visualization

### 8.7 Graph & Visualization

#### 8.7.1 Orchestration graph
- [ ] Implement tiddler relationship graph (questions → runs → answers)
- [ ] Add pipeline DAG visualization
- [ ] Create agent-tool relationship graph
- [ ] Implement lineage tracing visualization
- [ ] Add interactive graph filtering

---

## Phase 9: Advanced Features

### 9.1 Embeddings & RAG

#### 9.1.1 Embedding generation
- [ ] Implement embedding generation for tiddlers
- [ ] Add incremental embedding updates (on content change)
- [ ] Create embedding storage (fields or separate tiddlers)
- [ ] Implement embedding versioning
- [ ] Add embedding batch processing

#### 9.1.2 Vector search
- [ ] Implement similarity search over embeddings
- [ ] Add k-NN search
- [ ] Create semantic search UI
- [ ] Implement hybrid search (keyword + semantic)
- [ ] Add search result ranking

#### 9.1.3 RAG tool
- [ ] Create `SemanticSearch` tool for agents
- [ ] Implement context window management
- [ ] Add citation generation (link back to source tiddlers)
- [ ] Create RAG pipeline template
- [ ] Implement answer grounding (cite sources)

### 9.2 Multi-turn Reasoning

#### 9.2.1 Reasoning flows
- [ ] Implement "list sub-questions" step
- [ ] Create "answer sub-questions" step
- [ ] Implement "integrate answers" step
- [ ] Add reasoning graph tracking
- [ ] Create reasoning step editing/correction

### 9.3 Background Maintenance

#### 9.3.1 Declarative ops
- [ ] Define maintenance rule schema (filter, operation, schedule)
- [ ] Implement rule evaluation engine
- [ ] Add rule-triggered job creation
- [ ] Create rule status tracking
- [ ] Implement rule conflict detection

#### 9.3.2 Maintenance operations
- [ ] Implement "update summary" operation
- [ ] Implement "update tags" operation
- [ ] Implement "update embeddings" operation
- [ ] Implement "check freshness" operation (vs. Wikipedia)
- [ ] Add custom operation plugins

### 9.4 Advanced Pipelines

#### 9.4.1 Dynamic pipelines
- [ ] Implement conditional steps (if/else based on previous results)
- [ ] Add loop steps (iterate over results)
- [ ] Create pipeline composition (call pipelines from pipelines)
- [ ] Implement parallel step execution
- [ ] Add pipeline templates (reusable patterns)

### 9.5 Governance & Audit

#### 9.5.1 Change proposals
- [ ] Implement patch tiddler system (AI proposes, human reviews)
- [ ] Create diff view for patches
- [ ] Add patch approval workflow
- [ ] Implement patch application
- [ ] Create patch rejection logging

#### 9.5.2 Audit trail
- [ ] Implement AI change logging
- [ ] Add filter for "AI-modified tiddlers"
- [ ] Create audit report views
- [ ] Implement change attribution (which agent/run)
- [ ] Add rollback support for AI changes

---

## Phase 10: Polish & Production

### 10.1 Documentation

#### 10.1.1 User documentation
- [ ] Write user guide for AI features
- [ ] Create quick-start tutorial
- [ ] Add video walkthroughs
- [ ] Document common workflows
- [ ] Create troubleshooting guide

#### 10.1.2 Developer documentation
- [ ] Write plugin development guide
- [ ] Document provider API
- [ ] Create tool development guide
- [ ] Document pipeline step types
- [ ] Add API reference

#### 10.1.3 Example wikis
- [ ] Create "AI Starter" wiki edition
- [ ] Add example prompts and pipelines
- [ ] Create demo agents
- [ ] Add sample evaluation datasets
- [ ] Create showcase wiki with use cases

### 10.2 Testing

#### 10.2.1 Unit tests
- [ ] Write tests for core utilities (hashing, JSON, timestamps)
- [ ] Add tests for provider abstraction
- [ ] Create tests for queue system
- [ ] Implement tests for prompt rendering
- [ ] Add tests for pipeline execution

#### 10.2.2 Integration tests
- [ ] Write end-to-end question-to-answer tests
- [ ] Add pipeline execution tests
- [ ] Create agent conversation tests
- [ ] Implement eval run tests
- [ ] Add import pipeline tests

#### 10.2.3 Performance tests
- [ ] Benchmark queue throughput
- [ ] Test large batch sampling (100+ samples)
- [ ] Measure embedding generation performance
- [ ] Test concurrent run execution
- [ ] Profile memory usage

### 10.3 Error Handling & Resilience

#### 10.3.1 Error handling
- [ ] Implement comprehensive error types
- [ ] Add error recovery strategies
- [ ] Create error logging and reporting
- [ ] Implement user-friendly error messages
- [ ] Add error monitoring and alerts

#### 10.3.2 Resilience
- [ ] Implement circuit breakers for providers
- [ ] Add fallback providers
- [ ] Create graceful degradation (work offline)
- [ ] Implement data backup and restore
- [ ] Add disaster recovery procedures

### 10.4 Security Hardening

#### 10.4.1 Security review
- [ ] Audit API key storage and handling
- [ ] Review prompt injection vulnerabilities
- [ ] Check tool sandboxing
- [ ] Audit external data handling
- [ ] Review consent and privacy mechanisms

#### 10.4.2 Security features
- [ ] Implement API key encryption
- [ ] Add rate limiting per user
- [ ] Create content sanitization for external inputs
- [ ] Implement tool permission system
- [ ] Add security event logging

### 10.5 Performance Optimization

#### 10.5.1 Caching
- [ ] Implement run result caching (by hash)
- [ ] Add prompt rendering caching
- [ ] Create embedding caching
- [ ] Implement pipeline step caching
- [ ] Add TTL and cache invalidation

#### 10.5.2 Optimization
- [ ] Optimize tiddler queries (indexes, filters)
- [ ] Reduce memory footprint for large batches
- [ ] Optimize JSON field storage
- [ ] Implement lazy loading for large datasets
- [ ] Add pagination for large result sets

### 10.6 Deployment & Operations

#### 10.6.1 Deployment
- [ ] Create deployment scripts
- [ ] Add environment configuration (dev, staging, prod)
- [ ] Implement feature flags
- [ ] Create release checklist
- [ ] Add rollback procedures

#### 10.6.2 Monitoring
- [ ] Implement health checks
- [ ] Add metrics collection (runs, costs, latency)
- [ ] Create monitoring dashboards
- [ ] Add alerting (budget exceeded, errors, downtime)
- [ ] Implement log aggregation

---

## Phase 11: Community & Ecosystem

### 11.1 Plugin System

#### 11.1.1 Plugin architecture
- [ ] Define plugin interface for custom providers
- [ ] Create plugin interface for custom tools
- [ ] Add plugin interface for custom metrics
- [ ] Implement plugin interface for custom pipeline steps
- [ ] Create plugin discovery and loading

### 11.2 Sharing & Collaboration

#### 11.2.1 Export/import
- [ ] Implement prompt library export
- [ ] Create pipeline template export
- [ ] Add agent export
- [ ] Implement dataset export
- [ ] Create config bundle export/import

#### 11.2.2 Community templates
- [ ] Create prompt library repository
- [ ] Add pipeline template gallery
- [ ] Create agent marketplace
- [ ] Implement dataset sharing
- [ ] Add community showcase

### 11.3 Extensions

#### 11.3.1 Advanced integrations
- [ ] Add GitHub integration (issues, PRs, code)
- [ ] Create Slack/Discord notification integration
- [ ] Add email integration
- [ ] Implement webhook support
- [ ] Create Zapier/IFTTT integration

---

## Appendix: Cross-Cutting Concerns

### A. Accessibility
- [ ] Ensure keyboard navigation for all UI
- [ ] Add ARIA labels for screen readers
- [ ] Implement high-contrast mode support
- [ ] Add text alternatives for visualizations
- [ ] Test with accessibility tools

### B. Internationalization
- [ ] Extract all UI strings to language files
- [ ] Add translation infrastructure
- [ ] Support RTL languages
- [ ] Implement date/time localization
- [ ] Add number formatting localization

### C. Mobile Support
- [ ] Optimize UI for mobile screens
- [ ] Add touch gesture support
- [ ] Implement responsive layouts
- [ ] Test on mobile browsers
- [ ] Add progressive web app (PWA) support

### D. Backwards Compatibility
- [ ] Define migration path from pre-AI wikis
- [ ] Implement schema migration utilities
- [ ] Add backwards-compatible field handling
- [ ] Create data upgrade scripts
- [ ] Test with existing TiddlyWiki content

---

## Notes on Implementation Order

1. **Start with foundation (Phase 1)**: Get the schema, types, and utilities right before building on top.

2. **Iterative development**: Each phase can be developed incrementally. Start with minimal viable versions and iterate.

3. **Prioritize core flows**: Focus on the "standard question → sampling → meta-analysis → final answer" flow as the primary use case.

4. **Test early and often**: Build tests alongside features to catch issues early.

5. **UX feedback loops**: Prototype UIs early and get user feedback before building complex features.

6. **Security first**: Don't defer security to the end—build it in from the start.

7. **Performance matters**: Profile and optimize as you go, especially for batch operations.

8. **Document as you build**: Keep documentation up-to-date with the code to ease onboarding and maintenance.

---

## Success Metrics

To know if the implementation is successful, track:

- **Functionality**: All core features working as specified
- **Performance**: Batch of 100 runs completes in reasonable time (< 5 minutes for API, dependent on provider)
- **Reliability**: <1% error rate for runs under normal conditions
- **Usability**: Users can create questions and get answers without documentation
- **Cost efficiency**: Budget tracking prevents runaway costs
- **Security**: No exposed API keys, proper consent mechanism, safe tool execution
- **Extensibility**: New providers and tools can be added via plugins

---

## Conclusion

This implementation guide provides a comprehensive roadmap to turn the vision in `~Tiddly-AI.md` into reality. The phased approach allows for incremental development and testing, while the modular design ensures that components can be built, tested, and deployed independently.

Remember: **The goal is to make TiddlyWiki a powerful, transparent, and user-friendly AI orchestration control plane**—where the wiki itself becomes the IDE for LLM-powered workflows.
