## An introduction to
# [fit] **Session Types**
## by Wen Kokke

<!--!>

Overview:

- The untyped π-calculus
  + syntax
  + reduction semantics & structural congruence
  + an example
  + label transition semantics
  + commuting conversions considered harmful
  
- Translation from the λ-calculus to the π-calculus
  + functions are processes which receive their arguments,
    and send return values on a dedicated “output” channel
  + you’ve gotta be careful to insert some administrative reductions, 
    so that the translations of the various terms block according to the CBV reduction order

<---->

--------------------------------------------------------------------------------

# Roadmap

[.text: text-scale(0.95)]

- The _untyped λ-calculus_! **(So powerful, so scary…)**
- Taming the λ-calculus with types…
<br />
- The _untyped π-calculus_! **(Is even scarier…)**
- Taming the π-calculus with types…
<br />
- Concurrent λ-calculus! **(λ and π together…)**

^
I like to start my talks with a little bit of a roadmap, so we all know where we’re going.

^
First, we’re gonna talk about the λ-calculus. Y’all are probably pretty familiar with the λ-calculus, so I’m gonna be pretty brief. We’ll talk about the untyped λ-calculus, and how’ it’s *really* powerful, and *really* scary. We’ll talk about taming that scary power using types, and the difficulty of taming it without taking all the oomph out.

^
Then, we’ll talk about the π-calculus. We’ll take a bit more time, there, since a lot of y’all might not have seen the π-calculus before. Essentially, the λ-calculus studies functions, and the π-calculus studies message-passing processes. We’ll then talk about how the untyped π-calculus is even more powerful, and even scarier, and again we’ll talk about taming that scary power using types, and the problems that come with it.

^
Finally, we’ll talk about having the best of both worlds, in concurrent λ-calculus, where we have both higher-order functions *and* message-passing concurrency.

--------------------------------------------------------------------------------

# The untyped lambda calculus

$$
\begin{array}{c}
  \begin{array}{l}
  \text{Term} \; L, M, N
  \\
  \quad
    \begin{array}{rl}
    := & x 
    \;\mid\;    \lambda x.M 
    \;\mid\;    M \; N
    \end{array}
  \end{array}
  \\
  \\
  \begin{array}{l}
  (\lambda x.M)\;N
  \longrightarrow
  M\{N/x\}
  \end{array}
  \\
  \\
  \begin{array}{c}
  M
  \longrightarrow
  M^\prime
  \\ \hline
  M \; N
  \longrightarrow
  M^\prime \; N
  \end{array}
  \quad
  \begin{array}{c}
  N
  \longrightarrow
  N^\prime
  \\ \hline
  M \; N
  \longrightarrow
  M \; N^\prime
  \end{array}
\end{array}
$$

^
The untyped λ-calculus celebrated its 89th birthday last Monday, so to say that it’s been around for a while undersells it a bit.

^
It’s a pretty small system, and as such fits on a single slide.

^
There’s variables, λ-abstraction to make functions, and application to get rid of 'em.

^
There’s only one computation rule: if a function λx.M meets its argument N, we replace all occurrences of x in the function body M with the argument N.

^
(The other two reduction rules are just there to let us reduce underneath applications.)

^
The λ-calculus is *very* powerful—*some stuff about it being a “universal model of computation”*—but that power comes at the cost of also being able to express quite a lot of scary programs that do bad stuff.

[church1932]: https://www.jstor.org/stable/1968337

--------------------------------------------------------------------------------

# The untyped lambda calculus **is powerful**

$$
\begin{array}{lrl}
Y
& =               & \lambda f. (\lambda x. x \; x) (\lambda x. f \; (x \; x))
\end{array}
$$

$$
\begin{array}{lrl}
Y \; f
& \longrightarrow & f \; (Y \; f)
\\
& \longrightarrow & f \; (f \; (Y \; f))
\\
& \longrightarrow & f \; (f \; (f \; (Y \; f)))
\\
& \longrightarrow & \dots
\end{array}
$$

^
For instance, the λ-calculus comes with general recursion out of the box, via the Y combinator!

^
That’s good—as programmers, we like recursion! Really simplifies your programs, not having to write out the case for every possible input!

--------------------------------------------------------------------------------

# The untyped lambda calculus **is scary**

$$
\begin{array}{lrl}
Y
& =               & \lambda f. (\lambda x. x \; x) (\lambda x. f \; (x \; x))
\end{array}
$$

$$
\begin{array}{lrl}
\Omega 
& =               & (\lambda x. x \; x) (\lambda x. x \; x)
\\
& \longrightarrow & (x \; x)\{\lambda x. x \; x/x\}
\\
& =               & (\lambda x. x \; x) (\lambda x. x \; x)
\end{array}
$$

^
But pass Y the identity function, and you’ll get Ω, the program which runs forever, but never gets anything done!

^
That’s scary, I’d prefer not to have that!

--------------------------------------------------------------------------------

# The untyped lambda calculus **is scary**

If we want *more* than just functions…

<br />

$$
plus \; 1 \; true
$$

<br />

…we have to worry about silly stuff like this!

^
And if we want *more* than just functions… if we want some primitive types, like Booleans and integers… which, I know, big ask… we have to worry about silly stuff like this!

^
(Listen, I know this is all pretty familiar stuff, but it’s gonna be relevant by analogy!)

--------------------------------------------------------------------------------

# Taming the lambda calculus **with types**

$$
\begin{array}{c}
  \begin{array}{c}
  \Gamma \ni x : A
  \\ \hline
  \Gamma \vdash x : A
  \end{array}
  \quad
  \begin{array}{c}
  \Gamma, x : A \vdash M : B
  \\ \hline
  \Gamma \vdash \lambda x.M : A \to B
  \end{array}
  \\\\
  \begin{array}{c}
  \Gamma \vdash M : A \to B \quad \Gamma \vdash N : A
  \\ \hline
  \Gamma \vdash M \; N : B
  \end{array}
\end{array}
$$

^
Enter types! Here’s the simply-typed λ-calculus, which again fits on a single slide…

^
I’m hoping you’ve all seen this before, but if you haven’t:
• we check terms in some context Γ, which is just a bag of typing assignments
• a variable x has type A if there’s an assignment in the bag that says so
• if we’ve got something of type B which uses something of type A, we can abstract over that something to create a function from A → B
• if we’ve got something of type A → B, and something of type A, we can apply the one to the other to get something of type B

^
However, if you only allow yourself to use simply-typed programs… well, you get rid of all the scary, but also… kinda of all the power? Specifically, you lose recursion!

^
Queue the history of type theory, trying to make this system more permissive, while still keeping lots of the scary stuff out.

--------------------------------------------------------------------------------

# Taming the lambda calculus **with types**

$$
\begin{array}{c}
  \begin{array}{c}
  \\ \hline
  x : A \vdash x : A
  \end{array}
  \quad
  \begin{array}{c}
  \Gamma, x : A \vdash M : B
  \\ \hline
  \Gamma \vdash \lambda x.M : A \multimap B
  \end{array}
  \\\\
  \begin{array}{c}
  \Gamma \vdash M : A \multimap B \quad \Delta \vdash N : A
  \\ \hline
  \Gamma, \Delta \vdash M \; N : B
  \end{array}
\end{array}
$$

^
Let’s briefly talk about another type system for the λ-calculus: the linear λ-calculus.

^
What this changes is that it demands that every variable gets used exactly once. Every time you check an application, you have to decide which part of the bag goes to the function, and which part goes to the argument, and by the time you reach a variable, you must’ve split everything else off to be used in different parts of the term. Also you write this cute lollipop now, instead of the function arrow.

^
Essentially, what you’re left with here is the calculus of permutations. Think of lists… If you’re writing a function on lists, but you have to use every element in the list exactly once, what kinds of programs can you write? Permutations. That’s it.

^
This particular type system will be relevant later on!

--------------------------------------------------------------------------------

# Let’s talk pi calculus

^
Okay, let’s talk π-calculus.

^
The π-calculus is pretty young—it didn’t show up until 1992, though it’s heavily influenced by ideas dating back to the early 80s.

--------------------------------------------------------------------------------

# The untyped pi calculus — Syntax

$$
\begin{array}{l}
\text{Process}\;{P},{Q},{R}
\\
\quad
  \begin{array}{rll}
      :=    & (\nu {x}{x^\prime}){P} &\text{— create new channel }{x}{\leftrightarrow}{x^\prime}
  \\\mid    & ({P}\parallel{Q})      &\text{— put }P\text{ and }Q\text{ in parallel}
  \\\mid    & 0                      &\text{— done}
  \\\mid    & x\langle{y}\rangle.{P} &\text{— send }y\text{ on }x
  \\\mid    & x(y).{P}               &\text{— receive }y\text{ on }x
  \\\mid    & !{P}                   &\text{— replicate }P
  \end{array}
\end{array}
$$

^
It’s also pretty big! It’s got twice as many *things* in it as the λ-calculus.

^
Instead of functions, we’re talking about processes, which are built using *six* different constructors. 
• We’ve got ν-binders: (νxx’) creates a new channel, and names it’s two endpoints x and x’, which we can then use to communicate.
• We’ve got parallel composition: to let you know that two processes are running in parallel, we separate them with a a double bar "∥". 
• We’ve got nil, the process that is done.
• We’ve got “send”, which sends some value y on x, and then continues as P.
• We’ve got “receive”, which receives some value on x, names it y, and then continues as P.
• We’ve got replication…

--------------------------------------------------------------------------------

# The untyped pi calculus — Semantics?

$$
\begin{array}{c}
  \begin{array}{c}
  (\nu xx^\prime)(x\langle{y}\rangle.{P}\parallel x^\prime(z).{Q}) 
  \longrightarrow
  (\nu xx^\prime)({P}\parallel{Q}\{y/z\})
  \end{array}
  \\
  \\
  \begin{array}{c}
    {P}
    \longrightarrow
    {P}^\prime
    \\ \hline
    (\nu xx^\prime){P}
    \longrightarrow
    (\nu xx^\prime){P}^\prime
  \end{array}
  \\
  \begin{array}{c}
    {P}
    \longrightarrow
    {P}^\prime
    \\ \hline
    {P}\parallel{Q}
    \longrightarrow
    {P}^\prime\parallel{Q}
  \end{array}
  \quad
  \begin{array}{c}
    {Q}
    \longrightarrow
    {Q}^\prime
    \\ \hline
    {P}\parallel{Q}
    \longrightarrow
    {P}\parallel{Q}^\prime
  \end{array}
\end{array}
$$

--------------------------------------------------------------------------------

# The untyped pi calculus — Semantics?

How do we reduce…

$$
\begin{array}{c}
  (\nu xx^\prime)(x^\prime(z).{Q}\parallel x\langle{y}\rangle.{P})
  \longrightarrow
  \;??
\end{array}
$$

Can we apply…

$$
(\nu xx^\prime)(x\langle{y}\rangle.{P}\parallel x^\prime(z).{Q}) 
\longrightarrow
(\nu xx^\prime)({P}\parallel{Q}\{y/z\})
$$

No!

^
We forgot to tell the reduction relation about parallel composition!

--------------------------------------------------------------------------------

# The untyped pi calculus — Semantics

$$
\begin{array}{lrll}
  P \parallel Q
  & \equiv
  & Q \parallel P
  \\
  P \parallel (Q \parallel R)
  & \equiv
  & (P \parallel Q) \parallel R
  \\
  P \parallel 0
  & \equiv
  & P
  \\
  (\nu xx^\prime)(\nu yy^\prime)P
  & \equiv
  & (\nu yy^\prime)(\nu xx^\prime)P
  \\
  (\nu xx^\prime)(P \parallel Q)
  & \equiv
  & (\nu xx^\prime)P \parallel Q,
  & \text{if}\;x,x^\prime\not\in{Q}
  \\
  !{P}
  & \equiv
  & !{P}\parallel{P}
\end{array}
$$

$$
\begin{array}{c}
  P \equiv P^\prime \quad P^\prime \longrightarrow Q^\prime \quad Q^\prime \equiv Q
  \\ \hline
  P \longrightarrow Q
\end{array}
$$

--------------------------------------------------------------------------------

# The untyped pi calculus **is scary**

$$
\begin{array}{c}
  (\nu xx’)%
  \left(
  \begin{array}{l}
  x\langle{y_1}\rangle.{P_1}\parallel x^\prime(z_1).{Q_1}\parallel
  \\
  x\langle{y_2}\rangle.{P_2}\parallel x^\prime(z_2).{Q_2}
  \end{array}
  \right)
  \\
  \\
  \downarrow
  \\
  \\
  (\nu xx’)%
  \left(
  \begin{array}{l}
  {P_1}\parallel {Q_1}\{y_1/z_1\}\parallel
  \\
  {P_2}\parallel {Q_2}\{y_2/z_2\}
  \end{array}
  \right)
  \quad
  \text{or}
  \quad
  (\nu xx’)%
  \left(
  \begin{array}{l}
  {P_1}\parallel {Q_1}\{y_2/z_1\}\parallel
  \\
  {P_2}\parallel {Q_2}\{y_1/z_2\}
  \end{array}
  \right)
\end{array}
$$
