# ♟ Chess Engine Testing Guide

## Overview

This document explains the testing setup, tools, standards, and workflows used in the Chess Engine project.

The purpose of testing is to ensure:
- Correct implementation of chess rules
- Stable game state transitions
- Prevention of regressions
- Reliable move validation
- Maintainable code quality

---

# Testing Stack

| Purpose | Tool |
|---|---|
| Unit Testing | JUnit 5 |
| Mocking | Mockito |
| Code Coverage | JaCoCo |
| Build Tool | Maven |
| Continuous Integration | GitHub Actions |

---

# Prerequisites

Before running tests, ensure the following are installed:

| Software | Recommended Version |
|---|---|
| Java | JDK 21+ |
| Maven | 3.9+ |
| Git | Latest |

---

# Verify Installation

## Verify Java

```bash
java -version
```

Expected output:

```text
openjdk version "21"
```

---

## Verify Maven

```bash
mvn -version
```

Expected output:

```text
Apache Maven 3.9.x
```

---

# Project Structure

```text
src/
├── main/
│   └── java/com/chess/
│       ├── board/
│       ├── engine/
│       ├── game/
│       ├── pieces/
│       ├── rules/
│       └── util/
│
└── test/
    └── java/com/chess/
        ├── board/
        ├── engine/
        ├── integration/
        ├── pieces/
        └── rules/
```

---

# Installing Testing Dependencies

This project uses Maven for dependency management.

Add the following dependencies to `pom.xml`.

---

# JUnit 5

Used for:
- Unit testing
- Assertions
- Test lifecycle management

## Maven Dependency

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.2</version>
    <scope>test</scope>
</dependency>
```

---

# Mockito

Used for:
- Mocking dependencies
- Isolating units during testing

## Maven Dependency

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.11.0</version>
    <scope>test</scope>
</dependency>
```

---

# JaCoCo

Used for:
- Code coverage reports

## Maven Plugin

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>

    <executions>

        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>

        <execution>
            <id>report</id>
            <phase>test</phase>

            <goals>
                <goal>report</goal>
            </goals>
        </execution>

    </executions>
</plugin>
```

---

# Recommended Maven Configuration

Add the following to the `<build>` section of `pom.xml`.

```xml
<build>

    <plugins>

        <!-- JUnit 5 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.5</version>
        </plugin>

        <!-- JaCoCo -->
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.11</version>

            <executions>

                <execution>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>

                <execution>
                    <id>report</id>
                    <phase>test</phase>

                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>

            </executions>

        </plugin>

    </plugins>

</build>
```

---

# Running Tests

---

# Run All Tests

```bash
mvn test
```

This command:
- Compiles the project
- Runs all tests
- Generates test reports

---

# Run Specific Test Class

```bash
mvn test -Dtest=PawnTest
```

---

# Run Specific Test Method

```bash
mvn test -Dtest=PawnTest#shouldAllowPawnDoubleMoveWhenAtStartingPosition
```

---

# Generate Coverage Report

```bash
mvn clean test
```

Coverage report location:

```text
target/site/jacoco/index.html
```

Open the file in your browser to view coverage metrics.

---

# GitHub Actions CI Setup

Create the following file:

```text
.github/workflows/test.yml
```

---

# GitHub Actions Workflow

```yaml
name: Chess Engine Tests

on:
  push:
    branches:
      - main
      - develop

  pull_request:
    branches:
      - main
      - develop

jobs:

  test:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4

        with:
          distribution: temurin
          java-version: 21

      - name: Build Project
        run: mvn clean install

      - name: Run Tests
        run: mvn test
```

---

# Test Naming Convention

Use descriptive names.

## Format

```text
should<Action>When<Condition>()
```

---

# Examples

```java
shouldAllowPawnDoubleMoveWhenAtStartingPosition()

shouldRejectMoveWhenKingWouldRemainInCheck()

shouldAllowKingsideCastlingWhenPathIsClear()
```

---

# Testing Standards

---

# 1. One Behavior Per Test

Each test should validate ONE behavior only.

---

## ✅ Good Example

```java
@Test
void shouldAllowKnightToJumpOverPieces() {

}
```

---

## ❌ Bad Example

```java
@Test
void testKnightMovementAndCapturesAndCheckDetection() {

}
```

---

# 2. Arrange → Act → Assert Pattern

All tests should follow:

```text
Arrange
Act
Assert
```

---

# Example

```java
@Test
void shouldMovePawnForwardOneSquare() {

    // Arrange
    Board board = new Board();

    Pawn pawn = new Pawn(PieceColor.WHITE);

    Position start = new Position(6, 0);

    board.setPiece(start, pawn);

    // Act
    List<Move> moves =
        pawn.generateMoves(start, board);

    // Assert
    assertEquals(1, moves.size());
}
```

---

# Core Testing Areas

---

# 1. Board Tests

Verify:
- Board initialization
- Piece placement
- Piece movement
- Captures
- Board cloning

---

# 2. Piece Movement Tests

Each piece requires isolated movement tests.

---

## Pawn

Test:
- Single move
- Double move
- Captures
- Illegal backward movement
- Promotion trigger

---

## Rook

Test:
- Horizontal movement
- Vertical movement
- Collision blocking

---

## Knight

Test:
- L-shaped movement
- Jumping over pieces

---

## Bishop

Test:
- Diagonal movement
- Blocking pieces

---

## Queen

Test:
- Combined rook + bishop movement

---

## King

Test:
- Single-square movement
- Illegal movement into check

---

# 3. Move Validation Tests

Verify:
- Illegal moves rejected
- Pinned pieces restricted
- King safety enforced

---

# 4. Special Move Tests

---

## Castling

Verify:
- King-side castling
- Queen-side castling
- Invalid castling scenarios

### Invalid Cases
- King already moved
- Rook already moved
- Squares under attack
- Pieces between king and rook

---

## En Passant

Verify:
- Immediate capture allowed
- Delayed capture rejected

---

## Promotion

Verify:
- Promotion triggers correctly
- Correct replacement piece used

---

# 5. Check & Checkmate Tests

---

## Check Detection

Verify attacks from:
- Pawns
- Knights
- Bishops
- Rooks
- Queens

---

## Checkmate

Verify:
- King in check
- No legal escape moves

---

# 6. Draw Rule Tests

---

## Stalemate

Verify:
- No legal moves
- King NOT in check

---

## Insufficient Material

Verify:
- King vs King
- King + Bishop vs King
- King + Knight vs King

---

## Threefold Repetition

Verify:
- Same position repeated 3 times

---

## Fifty-Move Rule

Verify:
- 50 moves without:
  - pawn move
  - capture

---

# Integration Testing

Integration tests validate interactions between systems.

---

# Example Scenarios

| Scenario | Systems |
|---|---|
| Legal move execution | Generator + Validator + Executor |
| Castling | Rules + Validator + Executor |
| Checkmate | Validation + Detection |

---

# Regression Testing

Whenever a bug is fixed:

1. Add a dedicated regression test
2. Ensure the test reproduces the bug
3. Keep the test permanently

---

# Example Regression Test

```java
@Test
void shouldRejectCastlingThroughCheck() {

}
```

---

# FEN-Based Testing (Recommended)

Use FEN strings to reproduce board states.

## Example

```text
rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR
```

Benefits:
- Easier debugging
- Easier reproduction
- Smaller setup code

---

# Code Coverage Targets

| Area | Target |
|---|---|
| Engine Logic | 90%+ |
| Special Rules | 95%+ |
| Utility Classes | 80%+ |

---

# Pull Request Requirements

A PR cannot be merged unless:

- [ ] Tests pass
- [ ] New logic includes tests
- [ ] No regression failures
- [ ] Coverage threshold maintained
- [ ] Code reviewed

---

# Common Chess Engine Bugs

These areas require extra testing.

| Area | Risk |
|---|---|
| Castling through check | High |
| En passant exposing king | High |
| Pinned pieces | High |
| Double check | High |
| Promotion edge cases | Medium |
| Repetition tracking | Medium |

---

# Recommended IDE Setup

---

# IntelliJ IDEA

Recommended plugins:
- JUnit
- Maven
- SonarLint

Enable:
- Auto-import Maven dependencies
- Test coverage runner

---

# VS Code

Recommended extensions:
- Extension Pack for Java
- Test Runner for Java
- Maven for Java

---

# Definition of Done

A feature is considered complete when:

- Acceptance criteria pass
- Unit tests added
- Integration tests pass
- Regression tests added
- Coverage maintained
- Pull request approved

---

# Final Notes

The engine should prioritize:

1. Correctness
2. Maintainability
3. Testability

Performance optimizations come later.
