# SSL Hi-jack demo

Illustrate the process of creating the illusion of accessing a credible website.
The user interface seamlessly mimics that of a legitimate website, effectively concealing the fact
that it's interacting with a locally hosted backend system.
The illusion is achieved by generating a fake self-signed certificate for the domain of the target
HTTPS site and modifying the user's system to accept and trust the falsified certificate.
