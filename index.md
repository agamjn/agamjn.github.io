---
layout: page
permalink: /
---

<div class="about-wrapper">
  <div class="about-content">
    <p>I am an entrepreneur, developer and the founder of <a class="underline" href="https://tensorfuse.io/">Tensorfuse</a>.</p>

    <h5>Some things about me:</h5>
    <ul>
        <li>Grew up in a <a class="underline" href="https://en.wikipedia.org/wiki/Ashoknagar_(Madhya_Pradesh)">small town</a> in India.</li>
        <li>Did my engineering from <a class="underline" href="https://en.wikipedia.org/wiki/IIT_Roorkee">IIT Roorkee</a>.</li>
        <li>Was a part of the Y Combinator W24 batch.</li>
        <li>Previously worked on semiconductor devices & computer vision. Find my publications <a class="underline" href="https://scholar.google.com/citations?hl=en&user=PNfj9uUAAAAJ">here</a>.</li>
        <li>I keep travelling between San Francisco and Bangalore.</li>
        <li>Currently working on making ML infra more accessible and efficient for researchers and engineers.</li>
        <li>When I am not working, you’ll find me playing Badminton, Ukulele and writing.</li>
    </ul>

        <h5>Some things I believe:</h5>
    <ul>
        <li>One should be passionate about the problem and not the tools.
            <ul>
                <li>Programming, design, bio-engineering, sales, marketing etc. are mere tools.</li>
                <li>Most people get attached to these tools rather than the problem.</li>
                <li>Develop passion for solving hunger, poverty, climate change or whatever and learn the tools necessary to solve them for yourselves, your country and the world.</li>
            </ul>
        </li>
        <li>Consume less, create more
            <ul>
                <li>Consumption dopamine is cheap</li>
                <li>Dopamine from creation elevates</li>
            </ul>
        </li>
        <li>Life can be a quest or a race.
            <ul>
                <li>If you’re innately curious, you’ll make it a quest</li>
                <li>Else, you’ll always end up competing as if you’re part of a constant race.</li>
            </ul>
        </li>
    </ul>

  </div>

  <!-- <div class="about-image-container">
    <div class="about-image">
      <img src="/assets/images/agam_zoo.jpg" alt="agam" class="img-responsive">
    </div>
  </div> -->

</div>


<style>

  .underline {
    text-decoration: underline;
  }

  .about-wrapper {
    display: flex;
    flex-wrap: wrap;
    align-items: flex-start;
    gap: 2rem;
  }
  .about-content {
    flex: 1 1 100%;
  }
  .about-image-container {
    flex: 0 0 300px;
    display: flex;
    flex-direction: column;
    align-items: center;
    order: -1;
  }
  .about-image {
    width: 300px;
    height: 300px;
    border-radius: 50%;
    overflow: hidden;
  }
  .about-image img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  @media screen and (max-width: 768px) {
    .about-wrapper {
      flex-direction: column-reverse;
    }
    .about-image-container {
      margin-bottom: 1rem;
    }
  }
</style>
