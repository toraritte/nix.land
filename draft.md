## 1. Introdution

Nix "_(dolstra-thesis_1-introduction) is about getting computer programs from one machine to another—and having them still work when they get there. This is the problem of software deployment_", "_a part of the field of Software Configuration Management (SCM)_".

"_(dolstra-thesis_1.1-software-deployment) Software deployment is the problem of managing the distribution of software to end-user machines. That is, a developer has created some piece of software, and this ultimately has to end up on the machines of end-users. After the initial installation of the software, it might need to be upgraded or uninstalled._"

"_Presumably, the developer has tested the software and found it to work sufficiently well, so the challenge is to make sure that the software works just as well, i.e., the same, on the end-user machines. I will informally refer to this as **correct deployment**: given identical inputs, the software should behave the same on an end-user machine as on the developer machine._"

> FOOTNOTE
> All this is greatly oversimplified: "_(dolstra-thesis_1.1-software-deployment) First, in general there is no single “developer”. Second, there are usually several intermediaries between the developer and the end-user, such as a system administrator. However, for a discussion of the main issues this will suffice._)

"_(dolstra-thesis_1.1-software-deployment) This should be a simple problem. For instance, if the software consists of a set of files, then deployment should be a simple matter of copying those to the target machines. In practice, deployment turns out to be much harder. This has a number of causes. These fall into two broad categories: *environment issues* and *manageability issues*._"

> **Environment issues**
> 
> The first category is essentially about correctness. The software might make all sorts of demands about the *environment* in which it executes: that certain other software components are present in the system, that certain configuration files exist, that certain modifications were made to the Windows registry, and so on. If any of those environmental characteristics does not hold, then there is a possibility that the software does not work the same as it did on the developer machine. Some concrete issues are the following:
> 
> • A software component is almost never self-contained; rather, it *depends* on other components to do some work on its behalf. These are its dependencies. For correct deployment, it is necessary that all dependencies are identified. This identification is quite hard, however, as it is often difficult to *test* whether the dependency specification is complete. After all, if we forget to specify a dependency, we don’t discover that fact if the machine on which we are testing already happens to have the dependency installed.
> 
> • Dependencies are not just a runtime issue. To *build* a component in the first place we need certain dependencies (such as compilers), and these need not be the same as the runtime dependencies, although there may be some overlap. In general, deployment of the build-time dependencies is not an end-user issue, but it might be in *source-based* deployment scenarios; that is, when a component is deployed in source form.  This is common in the open source world.
> 
> • Dependencies also need to be *compatible* with what is expected by the referring component. In general, not all versions of a component will work. This is the case even in the presence of type-checked interfaces, since interfaces never give a full specification of the observable behaviour of a component. Also, components often exhibit build-time *variability*, meaning that they can be built with or without certain optional features, or with other parameters selected at build time. Even worse, the component might be dependent on a specific compiler, or on specific compilation options being used for its dependencies (e.g., for Application Binary Interface (ABI) compatibility).
> 
> • Even if all required dependencies are present, our component still has to *find* them, in order to actually establish a concrete composition of components. This is often a rather labour-intensive part of the deployment process. Examples include setting up the dynamic linker search path on Unix systems [160], or the CLASSPATH in the Java environment.
> 
> > [160] TIS Committee. Tool Interface Specification (TIS) Executable and Linking Format (ELF) Specification, Version 1.2. http://www.x86.org/ftp/manuals/tools/elf.pdf, May 1995.
> 
> • Components can depend on non-software artifacts, such as configuration files, user accounts, and so on. For instance, a component might keep state in a database that has to be initialised prior to its first use.
> 
> • Components can require certain hardware characteristics, such as a specific processor type or a video card. These are somewhat outside the scope of software deployment, since we can at most *check* for such properties, not *realise* them if they are missing.
> 
> • Finally, deployment can be a *distributed* problem. A component can depend on other components running on remote machines or as separate processes on the same machine. For instance, a typical multi-tier web service consists of an HTTP server, a server implementing the business logic, and a database server, possibly all running on different machines.
> 
> So we have two problems in deployment: we must *identify* what our component’s requirements on the environment are, and we must somehow *realise* those requirements in the target environment. Realisation might consist of installing dependencies, creating or modifying configuration files, starting remote processes, and so on.
> 
> **Manageability issues**
> 
> The second category is about our ability to properly manage the deployment process. There are all kinds of operations that we need to be able to perform, such as packaging, transferring, installing, upgrading, uninstalling, and answering various queries; i.e., we have to be able to support the *evolution* of a software system. All these operations require various bits of information, can be time-consuming, and if not done properly can lead to incorrect deployment. For example:
> 
> • When we uninstall a component, we have to know what steps to take to safely undo the installation, e.g., by deleting files and modifying configuration files. At the same time we must also take care never to remove any component still in use by some other part of the system.
> 
> • Likewise, when we perform a component upgrade, we should be careful not to over- write any part of any component that might induce a failure in another part of the system. This is the well-known *DLL hell*, where upgrading or installing one applica- tion can cause a failure in another application due to shared dynamic libraries. It has been observed that software systems often suffer from the seemingly inexplicable phenomenon of “bit rot,” i.e., that applications that worked initially stop working over time due to changes in the environment [26].
> 
> > [26] Anders Christensen and Tor Egge. Store — a system for handling third-party applications in a heteroge- neous computer environment. In Selected papers from the ICSE SCM-4 and SCM-5 Workshops on Soft- ware Configuration Management, number 1005 in Lecture Notes in Computer Science. Springer-Verlag, 1995.
> 
> • Administrators often want to perform queries such as “to what component does this file belong?”, “how much disk space will it take to install this component?”, “from what sources was this component built?”, and so on.
> 
> • Maintenance of a system means keeping the software up to date. There are many different policy choices that can be made. For instance, in a network, system administrators may want to push updates (such as security fixes) to all client machines periodically. On the other hand, if users are allowed to administer their own machines, it should be possible for them to select components individually.
> 
> • When we upgrade components, it is important to be able to *undo*, or *roll back* the effects of the upgrade, if the upgrade turns out to break important functionality. This requires both that we remember what the old configuration was, and that we have some way to reproduce the old configuration.
> 
> • In heterogeneous networks (i.e., consisting of many different types of machines), or in small environments (e.g., a home computer), it is not easy to stay up to date with software updates. In particular in the case of security fixes this is an important problem. So we need to know what software is in use, whether updates are available, and whether such updates should be performed.
> 
> • Components can often be deployed in both source and binary form. Binary pack- ages have to be built for each supported platform, and sometimes in several variants as well. For instance, the Linux kernel has thousands of build-time configuration options. This greatly increases the deployment effort, particularly if packaging and transfer of packages is a manual or semi-automatic process.
> 
> • Since components often have a huge amount of variability, we sometimes want to expose that variability to certain users. For instance, Linux distributors or system administrators typically want to make specific feature selections. A deployment system should support this.

(dolstra-thesis_1.1-software-deployment) END

> TODO_2022-03-27_1842 "_1.2. The state of the art_" introduces important concepts by bringing up other mainstream package managers and enumerating their flaws. I don't want to add extra cognitive load with these; people coming to Nix usually burned themselves with plenty of others (and Dolstra's thesis is more than 15 years old - no clue how much RPM has changed for example).
> So the task is: **extract the concepts from this section**.
>
> ADDENDUM: Albeit, some of the points made need to be demonstrated on something... (see figures 1.5 and 1.6)

> TODO_2022-03-27_1845 "_1.3. Motivation_" lists the issues with other package managers, but I think it would be better to turn them around: what the properties of an ideal package manager would be.
>
> ADDENDUM: Either way, the list items presuppose that these are issues with other package managers, so it would be prudent to give examples. Which brings us back to TODO_2022-03-27_1842 that these will be outdated at one point, and would create a similar section to 1.2. What if they list would just stand by itself as a golden ideal?
>
> ADDENDUM-2: The terms used in the list (e.g., inexact, incomplete deployment) are all defined in 1.2 (hence TODO_2022-03-27_1842)
>
> ADDENDUM-3: Make sure to read with a critical eye from the standpoint of newcomers. For example, explain what an atomic deployment is or what "_It is difficult to adapt components._" means (in fact, have no clue what it means exactly).

(dolstra-thesis_1.4. The Nix deployment system) START

The main idea of the Nix approach is to store software components in isolation from each other in a central component store, under path names that contain cryptographic hashes of all inputs involved in building the component, such as /nix/store/rwmfbhb2znwp...-firefox- 1.0.4. As I show in this thesis, this prevents undeclared dependencies and enables support for side-by-side existence of component versions and variants.

(dolstra-thesis_1.4. The Nix deployment system) END

(dolstra-thesis_1.5-Contributions) START

The core idea of Nix is the *purely functional deployment model*. "_In this model a binary component is uniquely defined by the declared inputs used to build the component. This enables arbitrary component instances to exist side by side._"

The main properties of Nix:

• "_The cryptographic hashing scheme used by the Nix component store prevents undeclared dependencies, giving us complete deployment. Furthermore it provides support for side-by-side existence of component versions and variants._"

> TODO_2022-03-27_1912 Make sure to mention that "_Nix component store_" is the longer version of "Nix store" - and explain what a component is. (See [What is a Nix expression in regard to Nix package management?](https://stackoverflow.com/questions/58243554/what-is-a-nix-expression-in-regard-to-nix-package-management) SO thread about what a "component" is and other definitions.)

• "_Isolation between components prevents interference._"

• "_Users can install software independently from each other, without requiring mutual trust relations. Components that are common between users are shared, i.e., stored only once._"

• "_Upgrades are atomic; there is no time window in which the system is in an inconsistent state._"

• "_Nix supports O(1)-time rollbacks to previous configurations._"

• "_Nix supports automatic garbage collection of unused components._"

• "_The Nix component language describes not just how to build individual components, but also *compositions*. The language is a lazy, purely functional language. This is a good basis for a component composition language, as it allows dependencies and variability to be expressed in an elegant and flexible way._"

• "_Nix has a *transparent source/binary deployment model* that marries the best parts of source deployment models such as the FreeBSD Ports Collection, and binary deployment models such as RPM. In essence, binary deployment is an automatic optimisation of source deployment._"

• "_Nix is *policy-free*; it provides *mechanisms* to implement various deployment policies, but does not enforce a specific one. Some policies described in this thesis are *channels* (in push and pull variants), *one-click installations*, and *pure source deployments*._"

> TODO_2022-03-27_1915 Not sure how long channels will be around. Maybe forever, but that's probably because of all the confusion. flox seems to be using them in a sane way, but look into it again.

• "_A purely functional model supports efficient component upgrades despite the fact that a change to a fundamental component can propagate massively through the dependency graph._"

> TODO_2022-03-27_1919 How to generate this dependency graph?

• "_Nix supports distributed multi-platform builds in a transparent manner, i.e., a remote build is indistinguishable from a local build from the perspective of the user._"

• _Nix provides a good basis for the implementation of a build farm, which supports continuous integration and portability testing. I describe a Nix build farm that has greatly improved manageability over other build farms and is integrated with *release management*, that is, it builds concrete installable components that can be deployed directly through Nix._"

• "_The use of Nix for software deployment extends naturally to the field of *service deployment*. Services are running processes that provide some useful facility to users, such as web servers. They consist of software components, static configuration and data, and mutable state. The first two aspects can be managed using Nix, and its advantages—such as rollbacks and side-by-side deployment—apply here as well._"

• "_Though Nix is typically used to build *large-grained* components (i.e., traditional packages), it can also be used to build *small-grained* components such as individual source files. When used in this way it is a superior alternative to build managers_"

The above may sound like a product feature list, which is not necessarily a good thing as it is always possible to add any feature to a system given enough *ad-hockery*. However, these contributions all follow from a small set of principles, foremost among them the purely functional model.

(dolstra-thesis_1.5-Contributions) END

> TODO_2022-03-27_1922 "1.6. Outline of this thesis" provides a condensed view with good summaries of topics, problems, Nix concepts, etc.so after the rest of the thesis is "delved", return here to see if there are pithier explanations/summaries that are easier to digest.

> TODO_2022-03-27_1923 "1.6 Outline of this thesis": There is a part at the end with lots of papers; look them up and read them.
> > Origins Parts of this thesis are adapted from earlier publications. Chapter 2, “An Overview of Nix” is very loosely based on the LISA ’04 paper “Nix: A Safe and PolicyFree System for Software Deployment”, co-authored with Merijn de Jonge and Eelco Visser [50]. Chapter 3, “Deployment as Memory Management” is based on Sections 3–6 of the ICSE 2004 paper “Imposing a Memory Management Discipline on Software Deployment”, written with Eelco Visser and Merijn de Jonge [52]. Chapter 6, “The Intensional Model” is based on the ASE 2005 paper “Secure Sharing Between Untrusted Users in a Transparent Source/Binary Deployment Model” [48]. Section 7.5 on patch deployment appeared as the CBSE 2005 paper “Efficient Upgrading in a Purely Functional Component Deployment Model” [47]. Chapter 9, “Service Deployment” is based on the SCM-12 paper “Service Configuration Management”, written with Martin Bravenboer and Eelco Visser [49]. Snippets of Section 10.2 were taken from the SCM-11 paper “Integrating Software Construction and Software Deployment” [46].

> TODO_2022-03-27_1924 "1.7. Notational conventions" - Keep in mind if something is not clear.

(dolstra-thesis_2.1. The Nix store) START

Nix is a system for software deployment. The term *component* will be used to denote the basic units of deployment. These are simply sets of files that implement some arbitrary functionality through an interface. In fact, Nix does not really care what a component actually is. As far as Nix is concerned a component is just a set of files in a file system. That is, we have a very weak notion of *component*, weaker even than the commonly used definition in [155]. (What we call a component typically corresponds to the ambiguous notion of a *package* in package management systems. Nix’s notion of components is discussed further in Section 3.1.)

> [155] Clemens Szyperski. Component technology—what, where, and how? In Proceedings of the 25th International Conference on Software Engineering (ICSE 2003), pages 684–693, May 2003.

> TODO_2022-03-27_2102 Again, see [What is a Nix expression in regard to Nix package management?](https://stackoverflow.com/questions/58243554/what-is-a-nix-expression-in-regard-to-nix-package-management) SO thread about what a "component" is and other definitions. See also [What are interesting, unique, and/or non-standard uses of Nix?](https://discourse.nixos.org/t/what-are-interesting-unique-and-or-non-standard-uses-of-nix/12977).

Nix stores components in a *component store*, also called the *Nix store*. The store is simply a designated directory in the file system, usually /nix/store. The entries in that directory are components (and some other auxiliary files discussed later). Each component is stored in *isolation*; no two components have the same file name in the store.

A subset of a typical Nix store is shown in Figure 2.1. It contains a number of applications—GNU Hello 2.1.1 (a toy program that prints “Hello World”, Subversion 1.1.4 (a version management system), and Subversion 1.2.0—along with some of their dependencies.  These components are not single files, but directory trees. For instance, Subversion consists of a directory called bin containing a program called svn, and a directory lib containing many more files belonging to the component. (For simplicity, many files and dependencies have been omitted from the example.) The arrows denote runtime dependencies between components, which will be described shortly.

The notable feature in Figure 2.1 is the names of the components in the Nix store, such as /nix/store/bwacc7a5c5n3...-hello-2.1.1. The initial part of the file names, e.g., bwacc7a5c5n3..., is what gives Nix the ability to prevent undeclared dependencies and component interference. It is a representation of the cryptographic hash of all inputs involved in building the component.

Cryptographic hash functions are hash functions that map an arbitrary-length input onto a fixed-length bit string. They have the property that they are collision-resistant: it is computationally infeasible to find two inputs that hash to the same value. Cryptographic hashes are discussed in more detail in Section 5.1. Nix uses 160-bit hashes represented in a [base-32 notation](https://en.wikipedia.org/wiki/Base32), so each hash is 32 characters long. In this thesis, hashes are abridged to ellipses (...) most of the time for reasons of legibility. The full path of a directory entry in the Nix store is referred to as its *store path*. An example of a full store path is:

        /nix/store/bwacc7a5c5n3qx37nz5drwcgd2lv89w6-hello-2.1.1

So the file bin/hello in that component has the full path

        /nix/store/bwacc7a5c5n3qx37nz5drwcgd2lv89w6-hello-2.1.1/bin/hello

The hash is computed over all inputs, including the following:

+ The sources of the components.

+ The script that performed the build.

+ Any arguments or environment variables [152] passed to the build script.

> [152] W. Richard Stevens and Stephen A. Rago. Advanced Programming in the UNIX Environment. AddisonWesley, second edition, June 2005.

+ All build time dependencies, typically including the compiler, linker, any libraries used at build time, standard Unix tools such as cp and tar, the shell, and so on.

> TODO_2022-03-27_2104 Have there been any changes to how the hash is computed?

Cryptographic hashes in store paths serve two main goals. They prevent interference between components, and they allow complete identification of dependencies. The lack of these two properties is the cause for the vast majority of correctness and flexibility problems faced by conventional deployment systems, as we saw in Section 1.2.

**Preventing interference** Since the hash is computed over all inputs to the build process of the component, any change, no matter how slight, will be reflected in the hash. This includes changes to the dependencies; the hash is computed recursively. Thus, the hash essentially provides a unique identifier for a configuration. If two component compositions differ in any way, they will occupy different paths in the store (except for dependencies that they have in common). Installation or uninstallation of a configuration therefore will not interfere with any other configuration.

For instance, in the Nix store in Figure 2.1 there are two versions of Subversion. They were built from different sources, and so their hashes differ. Additionally, Subversion 1.2.0 uses a newer version of the OpenSSL cryptographic library. This newer version of OpenSSL likewise exists in a path different from the old OpenSSL. Thus, even though installing a new Subversion entails installing a new OpenSSL, the old Subversion instance is not affected, since it continues to use the old OpenSSL.

Recursive hash computation causes changes to a component to “propagate” through the dependency graph. This is illustrated in Figure 2.3, which shows the hash components of the store paths computed for the Mozilla Firefox component (a web browser) and some of its build time dependencies, both before and after a change is made to one of those dependencies. (An edge from a node A to a node B denotes that the build result of A is an input to the build process of B.) Specifically, the GTK GUI library dependency is updated from version 2.2.4 to 2.4.13. That is, its source bundle (gtk+-2.2.4.tar.bz2 and gtk+-2.4.13.tar.bz2, respectively) changes. This change propagates through the dependency graph, causing different store paths to be computed for the GTK component and the Firefox component.  However, components that do not depend on GTK, such as Glibc (the GNU C Library), are unaffected.

An important point here is that upgrading only happens by rebuilding the component in question and all components that depend on it. We never perform a destructive upgrade.  Components never change after they have been built—they are marked as read-only in the file system. Assuming that the build process for a component is deterministic, this means that the hash identifies the contents of the components at all times, not only just after it has been built. Conversely, the build-time inputs determine the contents of the component.

Therefore we call this a purely functional model. In purely functional programming languages such as Haskell [135], as in mathematics, the result of a function call depends exclusively on the definition of the function and on the arguments. In Nix, the contents of a component depend exclusively on the build inputs. The advantage of a purely functional model is that we obtain strong guarantees about components, such as non-interference.

**Identifying dependencies** The use of cryptographic hashes in component file names also prevents the use of undeclared dependencies, which (as we saw in Section 1.2) is the major cause of incomplete deployment. The reason that incomplete dependency information occurs is that there is generally nothing that prevents a component from accessing another component, either at build time or at runtime. For instance, a line in a build script or Makefile such as

        gcc -o program main.c ui.c -lssl

which links a program consisting of two C modules against a library named ssl, causes the linker to search in a set of standard locations for a library called ssl.

> FOOTNOTE To be precise, libssl.a or libssl.so on Unix.

These standard locations typically include “global” system directories such as /usr/lib on Unix systems. If the library happens to exist in one of those directories, we have incurred a dependency.  However, there is nothing that forces us to include this dependency in the dependency specifications of the deployment system (e.g., in the RPM spec file of Figure 1.2).

At runtime we have the same problem. Since components can perform arbitrary I/O actions, they can load and use other components. For instance, if the library ssl used above is a dynamic library, then program will contain an entry in its dynamic linkage meta-information that causes the dynamic linker to search for the library when program is started. The dynamic linker similarly searches in global locations such as /lib and /usr/lib.

Of course, there are countless mechanisms other than static or dynamic linking that establish a component composition. Some examples are including C header files, importing Java classes at compile time, calling external programs found through the PATH environment variable, and loading help files at runtime.

The hashing scheme neatly prevents these problems by not storing components in global locations, but in isolation from each other. For instance, assuming that all components in the system are stored in the Nix store, the linker line

        gcc -o program main.c ui.c -lssl

will simply fail to find libssl. Unless the path to an OpenSSL instance (e.g., /nix/store/- 5jq6jgkamxjj...-openssl-0.9.7d) was explicitly fed into the build process and added to the linker’s search path, no such instance will be found by the linker.

(dolstra-thesis_2.1. The Nix store) END
