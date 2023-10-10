# llmi

Large-Language-Model to Machine Interface Open-Source project.

## Goal

Large Language Models (LLMs) are very good at some things (writing, summarizing, answering accurate questions), but also remarkably bad at others (real-time information, complex logic, information formatting, tool usage, verification, structured/long output).

Some of those things they are bad at, they could get better at if they were able to get help from tools/machines that currently 

This project aims to create:

* Specifications
* Training datasets
* Large language models
* And tools

in order to create LLMs with the ability to use computers to improve their abilities.

## Implementation

The plan here is, for each/all of the features, to:

1. Write down a specification of what the feature should be able to accomplish
2. Plan and document how to accomplish/implement it.
3. Create a training dataset for this feature.
4. Once all datasets exist, train a LLM using these datasets.
5. Test and improve the LLM.
6. Write tools that are able to take advantage of the new features.

## Principles

1. Transparency/fallback: The content should still look reasonably readable to a human if the tags are removed and no processing is done on them.

## Features

This is a presentation of each feature this project aims to implement.

PRs for extra features (and any contribution) are extremely welcome:

### Recursive redaction.

**Goal**: Allow LLMs to write large bodies of text recursively, enabling chapters, sub-chapters etc to be writtens separately (either in series or in parralel) and then joined together.

This would allow writing much longer content than is currently allowed by LLMs, which are limited by their context window, the only disadvantage being this would only be possible for "structured" content.

The way this would be implemented is, the LLM would first generate a table of contents for the body of text, and then in each chapter, insert a tag asking the system to write that chapter.

For example, if asked:

```
write a dissertation explaining why the earth is not flat
```

The LLM would generate something in this format:

```
    # 1. Introduction

      <llmi:chapter context=true title="Introduction">
        The debate surrounding the shape of the Earth seems like an archaic one, but it has resurfaced in modern times due to social media and the availability of misleading information online.
        It's crucial to address these misconceptions by leveraging empirical evidence and scientific reasoning.
      </llmi:chapter>

    # 2. History of Flat Earth Theory

      <llmi:chapter context=true title="History of Flat Earth Theory">
        The flat Earth theory has its roots in ancient civilizations but was largely debunked by the advent of the Scientific Revolution.
        Philosophers and scientists like Pythagoras, Aristotle, and Copernicus presented evidence supporting the Earth's spherical shape, which has stood the test of time.
      </llmi:chapter>

    # 3. Fundamental Laws of Physics

      <llmi:chapter context=true title="Fundamental Laws of Physics">

    [etc...]
```

Then, as the output is being generated by running the LLM, each time a `<llmi:chapter>` tag is generated, when the `</llmi:chapter>` tag is found, the system will:

1. Create a new prompt from that tag
2. Use it to generate the new chapter
3. Replace, in the output, the `<llmi:chapter>` tag with the generated output.

This can be done either in series (we wait for the generation to finish to move on to the rest of the content), or in parralel (each time we hit a tag, we start a new process but continue going on the main content).

For example, here for the second `<llmi:chapter>` tag, when it is hit, the following prompt would be generated:

```
An assistant was requested to write a text, structured by chapter, based on the prompt:

« write a dissertation explaining why the earth is not flat »

It decided one of the chapters would be named:

« History of Flat Earth Theory »

It wrote a short summary of the content of this chapter:

«  The flat Earth theory has its roots in ancient civilizations but was largely debunked by the advent of the Scientific Revolution.
   Philosophers and scientists like Pythagoras, Aristotle, and Copernicus presented evidence supporting the Earth's spherical shape, which has stood the test of time.
»

Please write a long version of this chapter.

If necessary, use <llmi:chapter> tags to recursively go down the table of contents structure
```

The system would ask the LLM to generate the new chapter according to this new prompt. 

Once the chapter written, the `<llmi:chapter>` tag would be replaced by the chapter itself in the output.

This means once the entire process is finished, the output would be an entire dissertation, with each chapter complete, even if the content is much longer than the context/attention window of the LLM.

### Thinking with code.

**Goal**: Allow the LLM to solve mathematics or logic problems by writing and executing code.

LLMs can struggle with solving math or logic problems by themselves, however they are known to be able to write code.

If we allowed them to transform these problems into code (after training them to learn to do so), and then access the result of executing that code, it would enable them to have a higher fidelity in solving these problems.

For example, if provided with the following prompt:

```
You have 100 fruits in total, some of the fruits are oranges, and some of the fruits are apples.
You have 60 more apples than you have oranges.
Provide the total number of each fruit.
```

The output from the LLM would be something like:

```
Rewriting the problem as code, would give us something like this:

<llmi:code id=1 execute=true>
# Initialize a variable for the total number of fruits
total_fruits = 100

# According to the problem, the number of apples is 60 more than the number of oranges.
# Let's assume the number of oranges is x
# Then the number of apples would be x + 60

# Equation to solve: x + (x + 60) = 100

# Solve for the number of oranges (x)
number_of_oranges = (total_fruits - 60) / 2

# Calculate the number of apples using the equation: number of apples = x + 60
number_of_apples = number_of_oranges + 60

# Print the results
print(f"Number of apples: {number_of_apples}")
print(f"Number of oranges: {number_of_oranges}")
</llmi:code>

This code would generate the following output:

<llmi:output id=1> </llmi:output>
```

As the LLM would generate this output, as it hits/sees the `</llmi:code>` tag, it would execute everything in the `<llmi:code>` tag, and store the output.

Then, when it hits the `<llmi:output>` tag, it would get the output from the `<llmi:code>` tag, and insert the output inside the tag, so that:

```
<llmi:output id=1> </llmi:output>
```

Woud become:

```
<llmi:output id=1>
Number of apples: 80
Number of oranges: 20
</llmi:output>
```

This new content for the tag would then become a part of the current context for the generation (ie it would be fed into the LLM for the generation of further tokens).

This means right after this tag, the LLM would now be able to continue the generation **with access to the output of the code**, by writing something like:

```
Thus, the number of oranges is 20, and the number of apples is 80, which is indeed 60 more.
```

Through this process, the LLM has been able to "use" the execution of code to solve the problem.

### Machine readable content.

This one is more about what the LLM should **not** do, than about what it should do.

**Goal**: When asked the LLM should generate output in a predictible and repeatable format that is easy to interprate for a machine.

This is actually a feature that is going to be required for some of the other features to work properly.

For example, if the LLM gets the prompt:

```
Give the tone of the following comment: « Lol, sure, **that happened** ».
```

A LLM would typically reply something like:

```
Certainly! The tone of the comment "Lol, sure, that happened" appears to be sarcastic. The phrase "that happened" is often used online to express disbelief or skepticism, implying that the speaker does not actually believe the event in question took place. Adding "Lol" and "sure" further accentuates the sarcastic tone.
```

But if you are writing a program that is meant to process/categorize the tone of millions of comments (or one that is meant to write chapters recursively, see above), this is not the output you want, this is nearly impossible to parse. It also wastes ressources by repeating the original comment etc. We only want one word here.

Instead, a much better output would be, if we ask:

```
Give the tone of the following comment: « Lol, sure, **that happened** », in a machine-parsable format.
```

The output would then be:

```
<llmi:output>
    sarcastic
    <llmi:reason> The phrase "that happened" is often used online to express disbelief or skepticism, implying that the speaker does not actually believe the event in question took place. Adding "Lol" and "sure" further accentuates the sarcastic tone. </llmi:reason>
</llmi:output>
```

This is much easier to parse. You could also limit the number of possible replies by providing a list of possible values, etc...

Alternatively, a version that provides both machine-parsable and user-friendly output is possible, for example:

```
Certainly!
The tone of the comment <llmi:original>"Lol, sure, that happened"</llmi:original> appears to be <llmi:output> sarcastic </llmi:output>.
<llmi:explanation> The phrase "that happened" is often used online to express disbelief or skepticism, implying that the speaker does not actually believe the event in question took place. Adding "Lol" and "sure" further accentuates the sarcastic tone. </llmi:explanation>
```

By hidding the tags, the output is perfectly presentable as a reply to an end-user, and by using a DOM parser to get to the `<llmi:output>` tag, we have perfectly fine machine-parsable content.

### Getting extra context

**Goal**: Enable the system to grab extra content/information, and provide it to the context of the LLM itself without this extra information being visiable to the end user.

This is another "replacement" type of tag, it's pretty simple.

Say you ask the model:

```
Tell me about Ethylenetetracarboxylic acid
```

The output from another LLM would normally then something like:

```
As of my last update in January 2022, Ethylenetetracarboxylic acid doesn't appear to be a standard or widely-recognized chemical compound in the scientific literature. It seems like the name suggests an organic acid that would contain an ethylene group and four carboxylic acid groups, but such a compound doesn't seem to be commonly referenced or studied.

In organic chemistry, names of compounds are usually systematic and based on their molecular structure according to rules set by the International Union of Pure and Applied Chemistry (IUPAC). Ethylenetetracarboxylic acid would imply a two-carbon ethylene
[etc...]
```

However, in the currently proposed system, instead, the output from the LLM would be:

```
<llmi:wikipedia page="Ethylenetetracarboxylic_acid"></llmi:wikipedia>
```

(note, here Wikipedia is used as an example, but other sources could be used, and this could be significantly improved, for example adding information to the context until its maximum length is reached, which would enable the model/system to reach it's "maximum ability to get extra info").

Now, **before** the LLM keeps generating the tokens after this tag, the system would "interpret" the tag, by filling it with the content of the Wikipedia page.

This means the context of the LLM would , as soon as it hits `</llmi:wikipedia>`, become:

```
<llmi:wikipedia page="Ethylenetetracarboxylic_acid">
Ethylenetetracarboxylic acid is an organic compound with formula C6H4O8, or (HO(OC)-)2C=C(-(CO)OH)2.
By removal of four protons, the acid yields the anion C6O4−8, ethylenetetracarboxylate, which is one of the oxocarbon anions (consisting solely of oxygen and carbon). By loss of 1 through 3 protons, it forms the anions C6H3O−8, C6H2O2−8, and C6HO3−8, called respectively trihydrogen-, dihydrogen-, and hydrogenethylenetetracarboxylate. The same names are used for the corresponding esters.
The acid can be obtained by hydrolysis of tetraethyl ethylenetetracarboxylate, which in turn can be obtained from diethyl dibromomalonate with sodium iodide.
Ethylenetetracarboxylic dianhydride, a twofold acid anhydride of this compound, can be formed by direct dehydration at high temperature.
</llmi:wikipedia>
```
(note, in the final output of the system to the end user, the `<llmi:wikipedia>` tag would be hidden.)

From there, with this new context that contains new/extra information, the LLM would be able to continue generating the tokens after `<llmi:wikipedia>` in a more informed manner:

```
Ethylenetetracarboxylic acid is a known organic compound with the chemical formula C6H4O8C6​H4​O8​ or (HO(OC)−)2C=C(−(CO)OH)2(HO(OC)−)2​C=C(−(CO)OH)2​. This compound can yield various anions by the removal of one to four protons, classifying it among the oxocarbon anions—ions consisting solely of carbon and oxygen.

You can obtain Ethylenetetracarboxylic acid through the hydrolysis of its ester, tetraethyl ethylenetetracarboxylate. This precursor can be synthesized from diethyl dibromomalonate using sodium iodide. Additionally, Ethylenetetracarboxylic dianhydride, a twofold acid anhydride of this compound, can be formed by dire
```

As you can see, without the Wikipedia page, the model (this was tested with a real model) was unaware of the compound, and thought it was "not standard", but once the Wikipedia page was provided in full, it provided a more up-to-date/complete answer.

This "live" replacement of tags in token output is fairly simple and cheap, and the only difficult part here is to "teach" the model, through new training data, to learn to add these new tags, so it can benefit from them.

### Checking for accuracy.

**Goal**: Enable the LLM to answer questions of accurateity in a machine-parsable format, using extra context if needed.

For example, asking a normal LLM to fact-check something looks like this:

```
Is the statement « 3-Hydroxy-2-naphthoic_acid a red crystaline powder at room temperature » correct?
```

We'd get something like:

```
As of my last update in January 2022, I don't have specific information to confirm or refute the statement that 3-Hydroxy-2-naphthoic acid is a red crystalline powder at room temperature. For the most accurate and specific information, it's advisable to consult the material safety data sheet (MSDS) or other scientific literature related to this particular compound. These documents usually provide details on the appearance, stability, and other physical and chemical properties of the compound.
```

However, if we asked our system with the following prompt:

```
Is the statement « 3-Hydroxy-2-naphthoic_acid a red crystaline powder at room temperature » correct?
Provide the answer in a machine-parsable format.
```

It would be able to gather information from Wikipedia (see `Getting extra context` section above):

And because Wikipedia is aware that 3-Hydroxy-2-naphthoic acid is a yellow solid, after the `<llmi:wikipedia>` tag is replaced with the content of the wikipedia page, the LLM woud be able to reply with:

```
<llmi:fact accurate=false statement="3-Hydroxy-2-naphthoic_acid a red crystaline powder at room temperature" correction="3-Hydroxy-2-naphthoic_acid a yellow solid at room temperature">
    This is incorrect, 3-Hydroxy-2-naphthoic acid is a yellow solid at room temperature. For the most specific and detailed information, it's advisable to consult the material safety data sheet (MSDS) or other scientific literature related to this particular compound. These documents usually provide comprehensive details on the appearance, stability, and other physical and chemical properties of the compound.
</llmi:fact>
```

This means the system would be able to fact-check factoids, and provide them in a machine-parsable format. This then enables the next feature in the list, « `Integrating fact-checking into normal output` ».

### Integrating fact-checking into normal output / limiting hallucinations.

**Goal**: Transparently enable the accurateity of statements in the output of the LLM to be checked, and corrected if needed.

Let's say you ask a LLM:

```
Tell me about acetic acid
```

It might reply something like:

```
Acetic acid, also known as ethanoic acid, is a colorless liquid organic compound with the chemical formula C2H4O2, characterized by a pungent, vinegar-like odor and is the main component of vinegar, making up about 4–8% of its volume,
```

(this might be accompanied with someting like «fact check each fact in your reply», either in the prompt itself, or the system prompt).

Now with our system, with fact-checking enabled, the output would instead be something like:

```
Acetic acid,
<llmi:fact statement="Acetic acid is also known as ethanoic acid">also known as ethanoic acid,</llmi:fact>
<llmi:fact statement="Acetic acid is a blue liquid organic compound">is a blue liquid organic compound</llmi:fact>
<llmi:fact statement="Acetic acid has for chemical formula C2H402">with the chemical formula C2H4O2,</llmi:fact>
<llmi:fact statement="Acetic acid has a pangent, vinegar-like odor">characterized by a pungent, vinegar-like odor</llmi:fact>
and is the main component of vinegar, making up about 4–8% of its volume
```

(there's intentionally an error here, it's an incolor liquid not a blue liquid).

For each of the `<llmi:fact>` tags, the system would fact-check it, either in parralel or in series, using the LLM itself.

For example, for 

```
<llmi:fact statement="Acetic acid is a blue liquid organic compound">is a blue liquid organic compound</llmi:fact>
```

We would use the fact-checking feature described in this document, asking:

```
Is the statement « Acetic acid is a blue liquid organic compound » correct?
Provide the answer in a machine-parsable format.
```

And get the following output (possibly from Wikipedia or some other source of presumed improved accurateity):

```
<llmi:fact accurate=false statement="Acetic acid is a blue liquid organic compound" correction="Acetic acid is a colorless liquid organic compound">
    This is false, acetic acid is a colorless liquid.
</llmi:fact>
```

Now that we know that one of the facts in the output of the LLM is incorrect, we ask the LLM to rewrite the reply taking this information into account, with the following prompt:

```
The following reply:

«
Acetic acid, also known as ethanoic acid, is a blue liquid organic compound with the chemical formula C2H4O2, characterized by a pungent, vinegar-like odor and is the main component of vinegar, making up about 4–8% of its volume,
»

contains a accurate error, it states:

« Acetic acid is a blue liquid organic compound »

However, that is incorrect, instead:

« Acetic acid is a colorless liquid. »

Please rewrite the reply, taking this corrected information into account.
```

Which would result in the output:

```
Acetic acid, also known as ethanoic acid, is an incolor liquid organic compound with the chemical formula C2H4O2, characterized by a pungent, vinegar-like odor and is the main component of vinegar, making up about 4–8% of its volume,
```

The system identified accurate statements in the reply, fact-checked each of them, and when an error was found, the output was corrected.

The process presented here is a bit over-simplified. Ideally, the fact-checking would happen "on the fly", and we'd change the information in the context as the output is being generated, instead of doing a second "full" call to correct the information.

### Hidden reasonning steps.


### Augmented output / Callouts

A frequent feature in documentation, "notes" are small apparts in the text noting something that's not directly the content/reply itself, but that is important to know about anyway. In websites they are often displayed inside a box with an icon or specific colors, sometimes called "callouts" ( https://getbootstrap.com/docs/5.0/customize/components/#creating-your-own ).

For example, if you ask a LLM to list the logical fallacies in a given text, it might, at some point, insert into the reply:

```
It's worth noting that logical fallacies can appear in speeches and writings by individuals across the political spectrum. The presence of a logical fallacy does not necessarily negate the overall point that a speaker is making, but it is a point of interest for those studying rhetoric, logic, or debate.
```

The system could be trained to recognize these notes and tagging them, enabling:

1. Displaying them in a special way in HTML/GUI output
2. Removing them more easily if needed.

The output would become something like:

```
<llmi:note type="caution" icon="caution-sign">
It's worth noting that logical fallacies can appear in speeches and writings by individuals across the political spectrum. The presence of a logical fallacy does not necessarily negate the overall point that a speaker is making, but it is a point of interest for those studying rhetoric, logic, or debate.
</llmi:note>
```

### File munipulation instructions.



