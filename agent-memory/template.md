# Session file template

A standard session file template that supports recording multiple decision points.

> **Language note:** Write the whole file in the language the user is conversing in — unless the user explicitly asks for a specific language, in which case honor their choice. The English headings below show the structure; translate them along with the content when the user speaks another language. Do not leave the headings in English if the session itself is, say, Chinese.

---

# [Session topic]

- **Date**: YYYY-MM-DD HH:MM
- **Participants**: [user] + [AI tool name]
- **Type**: [coding / design / discussion / research / other]

## Background

[Briefly describe the background and goal of this session]

## Discussion points and decisions

### 1. [Decision point / topic 1]

**Problem**: [what needs to be solved]

**Options**:
- Option A: [description] — pros / cons
- Option B: [description] — pros / cons

**Decision**: [final choice]

**Rationale**: [why this was chosen]

---

### 2. [Decision point / topic 2]

**Problem**: [what needs to be solved]

**Decision**: [final choice]

**Rationale**: [why this was chosen]

---

(Add more decision points as needed...)

## Execution plan

[Describe the final plan, including key steps]

## Related content

### Code / file changes (if applicable)
- `path/to/file1`: [what changed]
- `path/to/file2`: [what changed]

### References
- [link or file reference]

## Lessons learned

- [lesson learned]
- [reusable pattern]
- [pitfall to avoid]

## Follow-ups / TODOs

- [ ] [follow-up task 1]
- [ ] [follow-up task 2]

---

## Filling guide

### Background
- Briefly explain why this discussion happened
- Make the goal explicit

### Discussion points and decisions
- Give each decision point its own section
- Record problem, options, decision, and rationale
- Add more detail if the decision process was complex

### Execution plan
- Describe the plan that was finally adopted
- Include key steps, but not excessive detail
- For coding tasks, note the main code changes

### Related content
- Record only key changes, don't copy entire files
- Link to relevant docs or resources

### Lessons learned
- Distill reusable knowledge
- Record pitfalls you hit
- Provide a reference for similar future problems

### Follow-ups / TODOs
- List tasks that need follow-up
- Makes progress easy to track

---

## Example: coding task

# Choosing a user-authentication approach

- **Date**: 2026-06-24 14:30
- **Participants**: developer + Claude Code
- **Type**: coding

## Background

The project needs user authentication, and the team is split between JWT and sessions.

## Discussion points and decisions

### 1. Auth approach

**Problem**: JWT or sessions?

**Options**:
- JWT: stateless, easy to scale, but can't be actively revoked
- Sessions: stateful, easy to manage, but need extra storage

**Decision**: JWT + refresh token

**Rationale**:
- The project is a separated front-end / back-end architecture
- Needs to support multi-device login
- Refresh tokens let us control session lifetime

---

### 2. Token storage

**Problem**: How should the front end store tokens?

**Options**:
- localStorage: simple, but vulnerable to XSS
- HttpOnly cookie: secure, but vulnerable to CSRF

**Decision**: HttpOnly cookie

**Rationale**: Security first; CSRF can be mitigated via the SameSite attribute

## Execution plan

1. Back end: implement JWT issuing and verification
2. Implement the refresh-token mechanism
3. Front end: store via HttpOnly cookie
4. Implement token invalidation on logout

## Related content

### Code / file changes
- `src/auth/jwt.ts`: JWT utility functions
- `src/auth/middleware.ts`: auth middleware
- `src/routes/auth.ts`: auth routes

### References
- [JWT best practices](https://jwt.io/introduction)
- [Cookie security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

## Lessons learned

- JWT fits separated front-end / back-end architectures
- Refresh tokens solve JWT's inability to be actively revoked
- HttpOnly + SameSite is the best practice for front-end token storage

## Follow-ups / TODOs

- [ ] Implement automatic token refresh
- [ ] Add login-device management
- [ ] Write tests for the auth module

---

## Example: non-coding task

# Technical proposal review — meeting minutes

- **Date**: 2026-06-24 10:00
- **Participants**: product manager + tech lead + AI assistant
- **Type**: discussion

## Background

Reviewing the technical proposal for a new feature to ensure feasibility and soundness.

## Discussion points and decisions

### 1. Database choice

**Problem**: MySQL or PostgreSQL?

**Options**:
- MySQL: mature and stable, rich ecosystem
- PostgreSQL: powerful features, great extensibility

**Decision**: PostgreSQL

**Rationale**:
- The project needs JSONB storage
- Needs full-text search
- The team has PostgreSQL experience

---

### 2. Caching strategy

**Problem**: How to design the cache?

**Options**:
- Redis: high performance, rich features
- Local cache: simple, no network overhead

**Decision**: Redis + local cache (two-tier)

**Rationale**:
- Hot data in the local cache
- Shared data in Redis
- Balances performance and consistency

## Execution plan

1. Stand up a PostgreSQL cluster
2. Deploy a Redis cluster
3. Implement the two-tier cache framework
4. Write the data-migration plan

## Lessons learned

- Tech selection should account for team experience
- Cache design must balance performance and consistency
- Proposal reviews should record the rationale behind decisions

## Follow-ups / TODOs

- [ ] Finish the database-migration plan
- [ ] Write the Redis usage guidelines
- [ ] Organize a tech share session
