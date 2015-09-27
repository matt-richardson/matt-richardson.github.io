---
layout: default
title: Contact Me
---

<div id="contact">
  <h1 class="page-title">Contact Me</h1>
  <div class="contactContent">
    <p class="intro">
    Feel free to drop me a note if you want to chat about anything. But please, dont send me 
    unsolicited commercial email aka spam.
    </p>
  </div>
  <form action="http://formspree.io/contact@mjrichardson.com" method="POST">
    <label for="name">Name</label>    
    <input type="text" id="name" name="name" class="full-width"><br>
    <label for="email">Email Address</label>
    <input type="email" id="email" name="_replyto" class="full-width"><br>
    <label for="message">Message</label>
    <textarea name="message" id="message" cols="30" rows="10" class="full-width"></textarea><br>
    <input type="hidden" name="_next" value="//mjrichardson.com/contact/thanks"/>
    <input type="hidden" name="_subject" value="New submission!" />
    <input type="text" name="_gotcha" style="display:none" />
    <input type="submit" value="Send" class="button">
  </form>
</div>