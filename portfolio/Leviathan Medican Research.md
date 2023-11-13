# Leviathan Medical Research

The following project showcases a simple example of creating files, folders, users, groups, and assigning permissions following the 'principle of least privilege.' 

## The Scenario

Leviathan Medical Research has a lot it needs to keep confidential: patient records and cutting-edge research projects. The company has dozens of doctors, hundreds of patients, and several projects that all need managing.

For this example, we'll focus on three doctors, three patients, and three projects.

#### Getting Started

While the terminal provided in the Google Cybersecurity Certificate is adequate for learning, I wanted something more robust for experimenting on my own. After doing some research on lightweight Linux distros that can be launched in a VM, I settled on [Ubuntu Server](https://ubuntu.com/download/server) run via [Multipass](https://multipass.run/). These instances of Ubuntu are run entirely with the CLI, don't hog system resources, and don't take up a lot of hard drive space. Best of all, I can experiment with all of the above without modifying my personal host OS (which is also Linux, running Mint).

After installing Multipass, starting an instance of Ubuntu is easy. The first line starts the VM, and the second line gives me access to the shell.

```bash
nick@carbon:~$ multipass launch -n leviathan
nick@carbon:~$ multipass shell leviathan
```

Now I am in the VM as user ubuntu:

```bash
ubuntu@leviathan:~$ whoami
ubuntu
```

Now I can begin setting up this fresh installation with files related to our example medical research company.

#### Creating Groups

I started by creating user four groups: doctors, and then the names of three research projects: merlin, nix, and odin.

```bash
ubuntu@leviathan:~$ sudo groupadd doctors
ubuntu@leviathan:~$ sudo groupadd merlin
ubuntu@leviathan:~$ sudo groupadd nix
ubuntu@leviathan:~$ sudo groupadd odin
```

#### Creating Users

I followed that by creating three users and assigning them to the above groups based on their necessary access levels:

* Dr. Reymond works on the Merlin and Nix projects.
* Dr. Osso works on the Merlin, Nix, and Odin projects.
* Dr. Malek works on the Nix project.


```bash
ubuntu@leviathan:~$ sudo useradd -G doctors,merlin,nix reymond
ubuntu@leviathan:~$ sudo useradd -G doctors,merlin,nix,odin osso
ubuntu@leviathan:~$ sudo useradd -G doctors,nix malek
```

#### Creating Files

Leviathan has hundreds of patients and these records need to be handled in such a way as to allow doctors to do their jobs, but also comply with HIPPA privacy standards. As such, only the doctor assigned to the patient is allowed access to read or write to the record. Additionally, to maintain privacy, patient names are not included in the filename, only their patient id number. If a patient is ever transfered to a different doctor, the new doctor gains ownership of the record.

```bash
ubuntu@leviathan:~$ cd ..
ubuntu@leviathan:/home$ mkdir records
ubuntu@leviathan:/home$ sudo chown ubuntu:doctors records
ubuntu@leviathan:/home$ chmod o-r,o-x records
ubuntu@leviathan:/home$ cd records
ubuntu@leviathan:/home/records$ touch patient_0.txt
ubuntu@leviathan:/home/records$ touch patient_1.txt
ubuntu@leviathan:/home/records$ touch patient_2.txt
ubuntu@leviathan:/home/records$ chmod g-r,o-r *.txt
ubuntu@leviathan:/home/records$ sudo chown reymond:doctors patient_0.txt
ubuntu@leviathan:/home/records$ sudo chown osso:doctors patient_1.txt
ubuntu@leviathan:/home/records$ sudo chown malek:doctors patient_2.txt
ubuntu@leviathan:/home/records$ ls -l
total 0
-rw------- 1 reymond doctors 0 Nov 12 14:18 patient_0.txt
-rw------- 1 osso    doctors 0 Nov 12 14:18 patient_1.txt
-rw------- 1 malek   doctors 0 Nov 12 14:18 patient_2.txt
```

Now we have three patients each assigned to their respective doctor, who is the owner of that record and the only one who can access the record. 

Next up are the research projects. Unlike the patients who are assigned to doctors 1-to-1, any numbner of doctors can belong to a research project. As such, each project also has a usergroup. Only the doctors who are assigned to the project group will have access to the files. Each project file belongs to the lead doctor and the project group. Since the projects are collaborative, anyone in the group has both read and write permissions. Whenever someone joins a team, they are added to the appropriate group. Likewise, whenever anyone leaves a team, they are removed from the appropriate group.

```bash
ubuntu@leviathan:/home$ cd ..
ubuntu@leviathan:/home$ mkdir projects
ubuntu@leviathan:/home$ sudo chown ubuntu:doctors projects
ubuntu@leviathan:/home$ chmod o-r,o-x projects
ubuntu@leviathan:/home$ cd projects
ubuntu@leviathan:/home/projects$ touch merlin.txt
ubuntu@leviathan:/home/projects$ touch nix.txt
ubuntu@leviathan:/home/projects$ touch odin.txt
ubuntu@leviathan:/home/projects$ chmod g+w,o-r *.txt
ubuntu@leviathan:/home/projects$ sudo chown reymond:merlin merlin.txt
ubuntu@leviathan:/home/projects$ sudo chown osso:nix nix.txt
ubuntu@leviathan:/home/projects$ sudo chown malek:odin odin.txt
ubuntu@leviathan:/home/projects$ ls -l
total 0
-rw-rw---- 1 reymond merlin 0 Nov 12 17:35 merlin.txt
-rw-rw---- 1 osso    nix    0 Nov 12 17:36 nix.txt
-rw-rw---- 1 malek   odin   0 Nov 12 17:36 odin.txt
```

Now Leviathan is ready to begin operations using the 'principle of least privilege': safely protecting patient data and medical research from unauthorized access, while allowing doctors to complete their work.

#### Wrapping Up

Once edits are made to the sample instance of Ubuntu, I can escape back to my host OS with Ctrl+D, then shut down the VM.

```bash
nick@carbon:~$ multipass stop leviathan
```

## Learnings

* Groups offer a lot of flexibility between permissions and users. It's a lot easier to assign permissions to groups first, then assign users to groups. This became particularly apparent as I thought about how to assign multiple users to a research project.

* Folder ownership was intentionally assigned to ubuntu (so that I did not need to sudo everything!) as well as the doctors group. This allows doctors to access the folders, but not necessarily all of the files in the folders, while keeping out users who do not need access to the folder. There might be a better way to approach this, such as assigning the folder back to root after I am done making modifications as ubuntu. This is something I will need to research further.
