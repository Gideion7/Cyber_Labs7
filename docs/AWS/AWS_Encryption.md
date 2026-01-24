# Encrypting Data at Rest with KMS 

## Project Overview

The goal wasn't just to "click the buttons" but to understand and implement a "defense-in-depth" strategy for data-at-rest. I used the AWS Key Management Service (KMS) as the central nervous system for encryption, applying it to data in S3 (Object Storage) and on an EBS Volume (Block Storage) for an EC2 Instance. Finally, I used CloudTrail to audit the entire process. 

## Tools & Services Used 
This project provided hands-on experience with the following core AWS services: 

- AWS Key Management Service (KMS): For creating, managing, and controlling our customer-managed encryption keys. 

- Amazon S3 (Simple Storage Service): For object storage and demonstrating server-side encryption (SSE-KMS) on data. 

- Amazon EC2 (Elastic Compute Cloud): For running the virtual server (LabInstance) whose root volume we encrypted. 

- Amazon EBS (Elastic Block Store): For managing the underlying block-level storage (the "virtual hard drive") for our EC2 instance. 

- AWS CloudTrail: For auditing and logging all API calls to provide a security "paper trail" of our actions.

  <img width="557" height="292" alt="image" src="https://github.com/user-attachments/assets/4d598ab8-7209-42fc-9f6e-4bf41a950ca7" />

## Task 1: Creating Our "Master Key" (The Bank Vault Key) 

Before I could encrypt anything, I needed a key. In AWS, the gold standard for this is a Customer Managed Key (CMK) in KMS.

1. Key Type: I chose Symmetric. This is a single, 256-bit key used for both encryption and decryption. 

2. Alias: I gave it a friendly name: MyKMSKey. 

3. Permissions: I defined who could manage the key (Key Administrators) and who could use the key (Key Users). For this lab, I assigned the voclabs role to both.

Analogy: The Bank Vault Key 
Think of a KMS Customer Managed Key (CMK) as a master key to a bank vault. This key is so secure that it never, ever leaves the bank (KMS). You can't see it or export it. You can only ask the bank manager (the KMS service) to use the key on your behalf to lock or unlock things. This is exactly what I did by creating MyKMSKey. 

<img width="604" height="377" alt="image" src="https://github.com/user-attachments/assets/a36089de-1f34-4448-b863-0549bbda0006" />

1.1 Selecting 'Symmetric' key type

<img width="543" height="304" alt="image" src="https://github.com/user-attachments/assets/df7298ac-b367-481e-9418-d128b2406ecc" />

1.2 Assigning the 'MyKMSKey' alias

<img width="598" height="471" alt="image" src="https://github.com/user-attachments/assets/d9bcec7d-f8b5-4e62-9778-39ac1bd0c3e3" />

1.3 Assigning the 'voclabs' role as Key User

<img width="478" height="259" alt="image" src="https://github.com/user-attachments/assets/1626be68-be97-4534-a780-22e95510f9bb" />

1.4 'MyKMSKey' listed and 'Enabled' in the console

## Task 2: Encrypting an S3 Object (Envelope Encryption in Action) 

Next, I wanted to protect a file. I used a simple clock.png image and an S3 bucket. 

What I did: 

1. I navigated to the S3 bucket (which had default encryption settings) and started the "Upload" process. 

2. I selected the clock.png file. 

3. Here's the crucial step: I expanded the Properties section, scrolled to Server-side encryption settings, and explicitly selected "AWS Key Management Service key (SSE-KMS)". 

4. From the dropdown, I chose MyKMSKey. 

5. I uploaded the file. 

Analogy: Envelope Encryption (The Safe Deposit Box) This is one of the most important concepts. How does S3 use my "bank vault key" (MyKMSKey) if that key can never leave the bank (KMS)? 

1. The Safe: When I uploaded clock.png, S3 generated a unique, one-time-use key to encrypt just that file. This is the Data Key. Think of it as a key to a small, new safe. 

2. The Problem: Now I have a safe (clock.png) and a key (the Data Key). I need to store this key somewhere! 

3. The Solution: I "asked" the bank manager (KMS) to lock this Data Key in a secure box. KMS used my MyKMSKey (the master vault key) to encrypt the Data Key. 

4. The Result: S3 stores the encrypted file, along with the encrypted data key. The plaintext data key is destroyed. 

My file is now wrapped in an "envelope" of encryption. To open it, I must first prove to KMS that I have permission to use MyKMSKey to unlock the key that unlocks the file. 

<img width="609" height="242" alt="image" src="https://github.com/user-attachments/assets/b715c3c7-b1fa-49bc-80c7-b3c231180380" />

2.1 Bucket's default encryption properties

<img width="624" height="421" alt="image" src="https://github.com/user-attachments/assets/ae5eb19f-04b4-4678-84ed-ebe4b36c052c" />

2.2 Selecting SSE-KMS and 'MyKMSKey' during upload

<img width="605" height="199" alt="image" src="https://github.com/user-attachments/assets/6e4d3d59-52e6-4aa1-b66d-a7f533a28723" />

2.3 Final object properties showing 'MyKMSKey' encryption

## Task 3: Testing Public Access (Failure as a Success!) 

This is my favorite part, as it highlights the power of "defense-in-depth." I grabbed the object's URL and tried to open it in a new browser tab.

***Failure 1: "Access Denied"** 

First, I got a simple AccessDenied error. 

- Why? This was the S3 Block Public Access setting working. It's the "bouncer at the club's front door" who didn't even let my request in. 

- What I did: I went into the bucket's Permissions tab, disabled "Block all public access," and then made the specific clock.png object "public using ACL."

<img width="627" height="176" alt="image" src="https://github.com/user-attachments/assets/317384f4-1711-47c5-a85b-e5df977eb713" />

3.1 The 'AccessDenied' XML error in a new tab

<img width="610" height="295" alt="image" src="https://github.com/user-attachments/assets/8623577a-fb97-4286-9dd4-765309666baf" />

3.2 Disabling 'Block all public access

<img width="610" height="260" alt="image" src="https://github.com/user-attachments/assets/b281c3dc-5b33-4133-8d33-ad86f9c92d66" />

3.3 Making the 'clock.png' object public via ACL

***Failure 2: InvalidArgument" (KMS.DisabledException)** 

Now that the "bouncer" was gone, I refreshed the page. I got a different error: InvalidArgument (Requests specifying Server Side Encryption with AWS KMS managed keys require AWS Signature Version 4). 

- Why? This is the magic! My request got in the door (S3 allowed it), but when S3 went to decrypt the file, it had to ask KMS. KMS saw the request was from an anonymous, unauthenticated public user and denied the decryption. 

- This is the core concept of defense-in-depth. My first security layer (S3 Public Access) failed (because I told it to), but my second, stronger layer (KMS key permissions) held firm. The safe was public, but it was still locked.

<img width="622" height="109" alt="image" src="https://github.com/user-attachments/assets/5ce643c8-4b0e-4f5a-9716-6017f080504f" />

3.4 The 'InvalidArgument' (KMS) XML error

## Task 4: Proving It Works (Authenticated Access) 

So, how do I see the file? I went back to the S3 console and, as my authenticated voclabs role, I selected clock.png and clicked Open. 

- Success! The image opened perfectly in a new tab. 

- Why? When I clicked "Open" from inside the console, S3 generated a "pre-signed URL." This is a special, temporary URL that includes my authentication (that "Signature Version 4" the error mentioned). 

- When S3 got this signed request, it went to KMS and said, "Hey, voclabs (a valid Key User) is asking to decrypt this." KMS checked its policy, approved the request, decrypted the data key, and S3 used it to decrypt the image. 

<img width="608" height="298" alt="image" src="https://github.com/user-attachments/assets/c6e97369-38d9-41cc-a4d2-bf034d400fc2" />

4.1 The 'clock.png' image loading correctly

<img width="637" height="75" alt="image" src="https://github.com/user-attachments/assets/94fb2ce4-1728-4fc1-906f-f9ef3d5e98dd" />

4.2 The long, pre-signed URL in the browser bar

## Task 5: The Audit Trail (Checking the Security Cameras) 

Everything I've done so far has been based on trust. How do I prove these API calls happened? I used CloudTrail. 

Analogy: The Security Camera Footage CloudTrail is the immutable, unchangeable security log for your entire AWS account. If an API call happens, it's recorded. It's the "security camera footage" that shows who entered the bank vault, when, and what they did. 

What I did: 

1. I went to the CloudTrail console and filtered the Event history by Event source: kms.amazonaws.com. 

2. I found the GenerateDataKey event. This was the "receipt" from Task 2, proving S3 (on my behalf) asked KMS to create a data key. 

3. I found the Decrypt event. This was the "receipt" from Task 4, proving S3 (on my behalf) asked KMS to decrypt that data key so I could view the image. 

This is critical for security and compliance. I can prove exactly when my key was used and by whom. 

<img width="613" height="422" alt="image" src="https://github.com/user-attachments/assets/2654f117-af38-475d-91d2-c72fd4ea059f" />

5.1  Filtering CloudTrail for 'kms.amazonaws.com

 <img width="556" height="468" alt="image" src="https://github.com/user-attachments/assets/21a412c5-925b-4b22-af4f-8f59cbaf6a4d" />

5.2 The 'GenerateDataKey' event record (JSON)

<img width="595" height="588" alt="image" src="https://github.com/user-attachments/assets/0d90f5be-94a1-454c-b587-24c1b172c1fc" />

5.3 The 'Decrypt' event record (JSON)

## Task 6: Encrypting an Existing EC2 Root Volume 

This was a more complex, real-world scenario. It's easy to encrypt a new server, but what about one that's already running? I encrypted the root volume (the "C: Drive" or "brain") of an existing LabInstance. 

Analogy: Server "Brain Transplant" You can't perform surgery on a running patient. The process I followed is the digital equivalent of a brain transplant. 

What I did (The 6-Step Process): 

1. Observe "Before": First, I confirmed the LabInstance's root volume was "Not encrypted." 

2. Stop Instance: I stopped the EC2 instance (the "patient" is now asleep). 

3. Snapshot: I detached the unencrypted root volume and created a snapshot (a perfect copy of the "brain"). 

4. Create Encrypted Volume: I used that snapshot to create a new volume. During this creation, I checked the "Encrypt this volume" box and selected MyKMSKey. 

5. Swap Volumes: I detached the old, unencrypted volume and attached my new, encrypted volume as the root device (/dev/xvda). 

6. Verify "After": I went back to the instance's Storage tab and confirmed its root volume was now listed as "Encrypted" and tied to MyKMSKey.

***A Note on Troubleshooting: The Availability Zone Mismatch**

I want to call out a problem I ran into during this step, as it highlights a fundamental concept of AWS architecture. 

- The Problem: When I got to Step 5 ("Swap Volumes"), I tried to attach my new encrypted volume, but the LabInstance was not on the list of available instances.

<img width="625" height="176" alt="image" src="https://github.com/user-attachments/assets/6159a31f-be8f-4190-85b3-fdf4c1bac39f" />

6.0 Before: Instance Storage tab showing 'Not encrypted

<img width="629" height="284" alt="image" src="https://github.com/user-attachments/assets/d8fb3373-3160-42e0-b45b-079e24ec754f" />

6.1 Snapshot 'Unencrypted Root Volume' status 'Completed'

<img width="641" height="485" alt="image" src="https://github.com/user-attachments/assets/d3017442-887e-497c-9abb-d536254e14ef" />

6.2 Create volume' dialog showing 'Encrypt' and 'MyKMSKey' selected in the correct AZ

<img width="641" height="335" alt="image" src="https://github.com/user-attachments/assets/83983021-de02-4b5b-a9de-cbdc8edece0a" />

6.3 Attach volume' dialog showing 'LabInstance' and '/dev/xvda

<img width="636" height="342" alt="image" src="https://github.com/user-attachments/assets/cf18a5d0-037c-428a-b85c-c62b1a15636c" />

6.4 Instance Storage tab showing 'Encrypted' and KMS Key ID

- The Troubleshooting: I was confused, so I double-checked my resources. 

1. I checked the LabInstance and saw it was in the Availability Zone (AZ) us-east-1c. 

2. I then checked the new encrypted volume I had just created and... I saw it was in us-east-1a. 

- The "Aha!" Moment: An EBS Volume (a "hard drive") and an EC2 Instance (a "server") must exist in the exact same physical data center (Availability Zone) to be attached. I had accidentally volume in the wrong zone! 

- The Fix: The solution was simple. I deleted the incorrect volume (the one in us-east-1a). I went back to my snapshot and re-created the volume, but this time I was careful to select the correct Availability Zone: us-east-1c. After that, the LabInstance appeared in the "Attach" list, and the process worked perfectly. 

This was a great practical lesson in why AZ-level architecture is so critical. 


## Task 7: The "Kill Switch" (Demonstrating Ultimate Control)

This was the final, most powerful test. What happens if my key is compromised or I need to instantly render all data useless in an emergency? I can disable the key. 

What I did: 

1. Disable Key: I went to the KMS console and disabled MyKMSKey. 

2. Test 1 (EC2): I tried to start LabInstance. It failed, going from "Pending" right back to "Stopped."
   - Why? The instance couldn't boot because EC2 couldn't get permission from KMS to decrypt the root volume. 

3. Test 2 (S3): I tried to open clock.png from the S3 console (which worked in Task 4). It failed with a KMS.DisabledException.
   - Why? Same reason. I was authenticated, but KMS flatly refused to use the disabled key. 

4. Proof (CloudTrail): I checked CloudTrail and found the CreateGrant event from EC2, which contained the error message: MyKMSKey is disabled. This was my "smoking gun" proof. 

5. The Fix: I went back and re-enabled MyKMSKey. 

6. Final Test: I started LabInstance again. This time, it booted up perfectly. 

This demonstrates that KMS is the central control plane. By disabling one key, I instantly and simultaneously rendered data in S3 and an EC2 volume completely inaccessible, even to authenticated users. 

<img width="605" height="425" alt="image" src="https://github.com/user-attachments/assets/62e02c63-cfdf-42c9-8429-3e9fdbe006be" />

7.1 Disabling 'MyKMSKey

<img width="631" height="284" alt="image" src="https://github.com/user-attachments/assets/2a3ef0ac-7d6b-4318-8cb8-2efe1af3a6a7" />

7.2 EC2 instance 'Stopped' after failed start attempt

<img width="639" height="91" alt="image" src="https://github.com/user-attachments/assets/29b16d59-afd8-454e-87c9-24c471f770c8" />

7.3  The 'KMS.DisabledException' error from S3

<img width="635" height="275" alt="image" src="https://github.com/user-attachments/assets/1afabc7f-1f17-4a7e-9dda-2fc48a1f234b" />

7.4 CloudTrail 'CreateGrant' event showing the 'key is disabled' error

<img width="634" height="115" alt="image" src="https://github.com/user-attachments/assets/198529c6-18b9-4e43-b73d-63526b720e42" />

7.5 After Enabling 'MyKMSKey' the 'LabInstance' status shwows 'Running' state

## Conclusion & Key Learnings

This project was a comprehensive tour of AWS's data-at-rest encryption strategy. 

My key takeaways are: 

- KMS is the Core: KMS is the central source of truth for encryption. Its policies and key states are the ultimate control. 

- Defense-in-Depth is Real: I proved that even with a "leaky" S3 bucket, KMS encryption provides a robust second line of defense that protects the data itself. 

- Auditability is Non-Negotiable: CloudTrail provides the essential, immutable log that proves who accessed what data (or key) and when. 

- Encryption is Flexible: I demonstrated how to apply encryption to both new S3 objects and, more complexly, to the root volume of an existing EC2 instance. 

- Troubleshooting is Learning: Making the Availability Zone mistake was a key learning moment. It cemented my understanding that AWS infrastructure is physically distributed and that these boundaries are not just suggestions—they are hard rules that impact architecture. 
