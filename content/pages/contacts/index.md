---
title: "Contact me"
template: "page"
socialImage: "./book.jpg"
---

I would love to hear from you! Whether you have a work offer, questions, or comments about my work, feel free to reach out. I'm always here to connect and collaborate.

You can contact me by filling out the form below or sending me an email directly at my email address. I try to respond as soon as possible, so you won't be left hanging.

For more information about my professional background and experience, please take a look at my curriculum vitae (CV) which you can download here:.

Let's connect and explore new opportunities together. I look forward to hearing from you!

import React from 'react';

const MyForm = () => (
  <form name="contact" method="POST" data-netlify="true">
    <input type="hidden" name="form-name" value="contact" />
    <div>
      <label htmlFor="name">Name</label>
      <input type="text" id="name" name="name" required />
    </div>
    <div>
      <label htmlFor="email">Email</label>
      <input type="email" id="email" name="email" required />
    </div>
    <div>
      <label htmlFor="message">Message</label>
      <textarea id="message" name="message" required></textarea>
    </div>
    <button type="submit">Submit</button>
  </form>
);

export default MyForm;