# Fundamental Mathematical Constants of the Universe

## Pi (π)
Pi is perhaps the most famous mathematical constant, representing the ratio of a circle's circumference to its diameter.

```mermaid
graph LR
    A[Circle] --> B[Circumference = 2πr]
    A --> C[Area = πr²]
    A --> D[π ≈ 3.14159...]
    
    subgraph Properties
    E[Irrational Number]
    F[Transcendental Number]
    G[Infinite Non-repeating Decimal]
    end
```

```mermaid
pie
    title "First few digits of π"
    "3" : 1
    "1" : 1
    "4" : 1
    "1" : 1
    "5" : 1
    "9" : 1
```

## Fibonacci Sequence
A sequence where each number is the sum of the two preceding ones, starting from 0 and 1.

```mermaid
graph LR
    A[0] --> B[1]
    B --> C[1]
    C --> D[2]
    D --> E[3]
    E --> F[5]
    F --> G[8]
    G --> H[13]
    H --> I[21]
    
    subgraph Pattern
    J["Fn = Fn-1 + Fn-2"]
    end
```

```mermaid
flowchart TD
    A[Start] --> B[n = 0, 1]
    B --> C[Next number = Sum of previous two]
    C --> D[0,1,1,2,3,5,8,13,21...]
    D --> E[Appears in nature]
    E --> F[Spiral patterns]
    E --> G[Plant growth]
    E --> H[Shell formations]
```

## Golden Ratio (φ)
The golden ratio (approximately 1.618033988749895) is found by dividing a line into two parts where the longer part divided by the smaller part equals the whole length divided by the longer part.

```mermaid
graph TD
    A[Golden Ratio φ] --> B["φ = (1 + √5) / 2"]
    A --> C[≈ 1.618033988749895]
    
    subgraph Applications
    D[Art & Architecture]
    E[Nature]
    F[Design]
    end
    
    subgraph Relationship to Fibonacci
    G["Ratio of consecutive Fibonacci numbers approaches φ"]
    end
```

## Euler's Number (e)
A fundamental mathematical constant approximately equal to 2.71828, basis of natural logarithms.

```mermaid
graph LR
    A[Euler's Number e] --> B["e = lim(n→∞)(1 + 1/n)^n"]
    A --> C[≈ 2.71828...]
    
    subgraph Applications
    D[Compound Interest]
    E[Natural Growth]
    F[Exponential Functions]
    end
```

```mermaid
flowchart TD
    A[e] --> B[Natural Growth]
    B --> C[Population Growth]
    B --> D[Radioactive Decay]
    B --> E[Compound Interest]
```

## Fine Structure Constant (α)
A fundamental physical constant characterizing the strength of the electromagnetic interaction.

```mermaid
graph TD
    A[Fine Structure Constant α] --> B["α = e²/ℏc"]
    A --> C[≈ 1/137.036]
    
    subgraph Components
    D[e = Elementary charge]
    E[ℏ = Reduced Planck constant]
    F[c = Speed of light]
    end
    
    subgraph Significance
    G[Quantum Electrodynamics]
    H[Atomic Structure]
    I[Electromagnetic Interactions]
    end
```
