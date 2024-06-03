
<link
  href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css"
  rel="stylesheet"
/>

```mermaid
---
title: Key Exchange
---
flowchart TD
    id[Key] --> subroutine[[subroutine]] --> cond{Cond}


    Start --> Stop --> data[(Database)]
    round(Rounded) --> id1([This is the text in the box]) --> circ((Circle))
     
    circ --yes --> id

    thick ==> Link
    thick ----> k[longer]
    dotted -.Link.-> With_text

    Invisible ~~~ Positioning


    Link --> a --> b & c & d

    g & e --> u & f

```

```mermaid
---
title: subgraphs
---
flowchart TB
    R[fa:fa-twitter] -->
    c1-->a2
    subgraph one
    a1-->F[fa:fa-key]
    end
    subgraph two
    b1-->b2
    end
    subgraph three
    c1-->c2
    end
```

<script>
  window.callback = function () {
    alert('A callback was triggered');
  };
</script>

```mermaid
flowchart LR
    O --> P 
    click O "https://www.github.com" _blank
    click P call callback() "Tooltip for a callback"
```

<body>
  <pre class="mermaid">
    flowchart LR
        A-->B
        B-->C
        C-->D
        click A callback "Tooltip"
        click B "https://www.github.com" "This is a link"
        click C call callback() "Tooltip"
        click D href "https://www.github.com" "This is a link"
  </pre>

  <script>
    window.callback = function () {
      alert('A callback was triggered');
    };
    const config = {
      startOnLoad: true,
      flowchart: { useMaxWidth: true, htmlLabels: true, curve: 'cardinal' },
      securityLevel: 'loose',
    };
    mermaid.initialize(config);
  </script>
</body>