---
layout: page
title: Contact
permalink: /contact/
fa: fa-envelope-o
---
<form action="https://getsimpleform.com/messages?form_api_token=e8b66bf8f8b9121a72951f763374d2f4" method="post">
	<input type="hidden" name="redirect_to" value="{{ site.url }}/thank-you/" />
	<label for="name">Name</label>
	<br />
	<input type="text" id="name" name="name" placeholder="Your Name" />
	<br />
	<label for="email">Email</label>
	<br />
	<input type="text" id="email" name="email" placeholder="Your Email" />
	<br />
	<label for="email">Message</label>
	<br />
	<textarea id="message" name="message" placeholder="Message" rows="8" cols="50"></textarea>
	<br />
	<input type="submit" value="Submit" />
</form>
