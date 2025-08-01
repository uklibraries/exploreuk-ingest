This document is about how we *currently* add content to [ExploreUK](https://exploreuk.uky.edu). ExploreUK as it currently exists was designed with the idea that packages would be prepared specifically for it, guaranteeing certain directory organization and file types, but with limited accompanying metadata, causing the need for extensive automatic analysis.

As we increase the rate at which we add "born-digital" packages, which were not prepared with submission into ExploreUK in mind, it is increasingly important that we have ways to move beyond *convention* and provide content managers with the tools needed to manipulate ingested items *directly*.

Note: this is not an instruction guide. It is an overview.

Here are the major stages of ExploreUK ingest.

1. Package preparation and submission (which intersects with KUDL)
2. Prepublication
3. Approval
4. Publication

Before describing the ingest process, we first sketch the basic picture of *things* in ExploreUK.
# Things in ExploreUK

This section is adapted from the [wiki page on ExploreUK metadata](https://wiki.ukpdp.org/wiki/ExploreUK/Metadata).

A single package being submitted to ExploreUK will usually represent multiple *things*. A thing can be a single physical entity, such as a photograph or a page of a manuscript; a physical entity with further structure, such as an issue of a newspaper; an abstract intellectual document, such as a finding aid; or an abstract assemblage of other things, such as a collection.

Things are represented in ExploreUK by means of *objects*. An object is an abstraction that represents a thing, possibly in multiple manifestations. For example, a page of a book can be manifested as:

* a photographic image of the page
* a printable PDF of the page
* a text transcription of the page
* an audio recording of someone reading the page aloud
* an XML file which records bounding boxes of individual words on the page.

An object can reference multiple such manifestations. It must also include sufficient metadata to enable it to be found through an ExploreUK search.

Some objects, such as pages of a book, are essentially subordinate to other objects. We generally don't want such objects appearing in top-level search results. Instead, a top-level search will surface a book, and selecting the book will "search inside" the book and identify appropriate pages. For historical reasons, objects that can appear in top-level search results are sometimes called *compound objects*.

We currently recognize three object types, the third of which has four subtypes.

* Collection - a hierarchically-organized set of Sections
* Section - a sequentially-organized set of Leaves
* Leaf - an object which represents a set of manifestations of a single item:
	* Audio - an audio file
	* Video - a video file
	* Image - an image, normally without text
	* Page - an image, normally with text.

Everything but *pages* may appear in top-level searches, that is, pages are the only objects which are not considered "compound objects".

A package to be added to ExploreUK must be analyzed into objects in order to be ingested. A *fundamental difficulty* is that insufficient explicit metadata is generally provided for this purpose on submission, and automatic analysis following various conventions is required to make ExploreUK ingest viable.

Some parts of the automatic analysis of ExploreUK packages appear late in the ingest process. They are not directly accessible to content managers. This means that when things go wrong, we often need to restructure and reprocess the submitted package instead of editing the structure directly.
# ExploreUK ingest

We currently track ExploreUK ingest with the somewhat anachronistically-named Trello board [KDL ingest requests](https://trello.com/b/Y1khx75h/kdl-ingest-requests). For another perspective on ExploreUK ingest, please consult the [Ingest Workflows Google Doc](https://docs.google.com/document/d/1xq8IGNPNvuj8-1bfU-XSF9eLt_0QIfu4MJMVK_ux-5U/).

## Package preparation and submission

Trello lists involved:

* Templates
* To Do
* To bag / Verify bag
* Ensure identifiers
* Submit to KUDL
* Split AIPs

How or whether files are digitized or otherwise generated before being submitted to ExploreUK is out of scope for this document. For the following discussion, we will primarily focus on a package being submitted for new ingest into ExploreUK.

Such a package must take the form of a BagIt bag. For more information, consult the [BagIt spec](https://www.rfc-editor.org/rfc/rfc8493.html). For packages with finding aids, submitters will process the EAD through a local system called Gossamer (a drop directory that provides output in a nearby directory) to generate an appropriate METS file and a directory hierarchy for digitized files. All of these files will go into the bag.

Submitters make a fresh card on To Do using an appropriate template. The current main template requires the submitter to make a series of edits to a default description:

```
- Finding aid or METS?
    
- Format: ____ (EAD format options are audio, audiovisual, images, maps, archival material)
    
- new material or reprocess? (add REPROCESS label) -- remove this whole line regardless
    
- replaces content at ____
    
- Bagged on pelican ____
    
- multiple items in bag?
    
- OCR needed?
    
- Approval: _____ (should be @ name) (always include this)
```

Note that the submitter is required to make decisions at multiple levels of abstraction:

* Is a finding aid included? (The system can detect this automatically, so why do we ask?)
* What basic sort of material is included? (Can this be inferred from metadata actually in the bag? Note that this implies that the material format is consistent throughout, which is increasingly not the case.)
* Is this new? If not, what was the old identifier in ExploreUK?
* Where can the submission files be found?
* How should the submission be processed? (Is there a reason not to generate OCR for any textual materials? The historical reasoning is likely that manuscripts can be difficult to read, making OCR a challenge.)
* Who is responsible for the correctness of this item in ExploreUK?

When a card on To Do is ready to submit (there is a potential ambiguity here—it takes time to prepare a card, and it is not immediately obvious whether a card is "ready" or still in preparation), a repository manager moves the card to the To bag / Verify bag list, and runs a script which currently performs the following validations:

* Ingest type is specified
* Need for OCR or the lack of it is unambiguous
* Reprocess item indicates previous location
* Package is clearly a batch or not a batch
* Approval is specified
* Bag directory exists
* (finding aid) METS file exists in standard location.
* (finding aid) METS file is valid.
* (finding aid) METS file marks EAD location.
* (finding aid) EAD exists in marked location.
* (finding aid) EAD is valid.
* (free_form / multipage) A unique candidate for METS file exists.
* (free_form / multipage) METS file is valid.
* Bag is valid

(Note: if the package is clearly a batch, the validator should, but does not, check that the "splittable AIPs" checklists have been added to the card. Or better yet, it should add them automatically.)

If any of these validations fail, the submission is invalid, and corrections must be made. Frequently, the submitter and repository manager must coordinate these corrections. If all of these succeed, the package is presumptively valid (though this presumption may later be proven false) and the card is moved to Ensure identifiers.

The purpose of the Ensure identifiers step is to interact with the Identity service to generate AIP and DIP identifiers—AIP for KUDL and DIP for ExploreUK—so that automatic services can consistently identify which package is being processed at any given time. This step is currently semi-manual: a repository manager uses some scripts to interact with the Identity service, and adds the identifiers to the description:

```
Identifiers:

- SIP: mets_G3954_L4_2A8_1919_A8
    
- AIP: xt7x0k26f94p
    
- DIP: xt7s7h1dp21k
```

This step could be largely automated once the Identity service is migrated from its current host.

After ensuring identifiers, a repository manager moves the card to Submit to KUDL. A script (which could run automatically) transfers the bag from a staging area to the KUDL digital preservation repository. Once this has been done, the resulting AIP is re-validated, and:

* If the card is marked "dark archive", the card is moved to Cleanup.
* If the card is marked as having multiple items, it is moved to Split AIPs.
* Otherwise, the card is moved to Create test DIP.

In any case, this ends the process of *KUDL* ingest. Now we can move to *ExploreUK* ingest.
## Prepublication

Trello lists involved:

* Metadata repairs (not frequently used)
* Create test DIP
* Index into test

The prepublication process consists of the following steps:

1. *(Create DIP shell)* Analyze an AIP and construct a DIP shell, which includes the METS file—and if appropriate, the EAD file—and a set of *instructions* for file transformations that must be performed (for example, converting a TIFF to a JPEG) in order to produce derivative files suitable to serve over the web.
2. *(Generate derivatives)* Build the DIP proper by *generating the derivatives* specified by the instructions in the DIP shell.
3. *(Index into test—steps 3 through 5)* Analyze the DIP to generate a MIP, which includes JSON files representing individual documents to be added to or updated in the Solr index.
4. (For finding aids) analyze the METS and EAD in the DIP to generate a page to represent that finding aid in the finding aid viewer.
5. Index the documents from the MIP into the Solr index.

The METS file in the DIP *implies* a certain structure to the objects described within, and the JSON files in the MIP *implicitly reference* this structure. This enables ExploreUK to behave as though this structure were actually explicitly represented directly. The term *DIP-MIP flip* refers to updating the artifacts we produce (and how we produce them) so that the structure is recorded *explicitly*, opening a path for tools to be provided to content managers for rich editing of objects in ExploreUK.

Now, here are some more technical details on the current process.
### Create test DIP

A card on the Create test DIP list is not automatically processed simply by virtue of being on that list. Instead, a script is used to generate manifests for each DIP to be ingested. This manifest is generated by extracting all pertinent metadata from the card on Trello.

Automatic processes generally reference the manifest, not Trello, to determine the status of a card, and update the manifest *before* updating Trello to record changes in the status. An exception to this is the approval step: automatic processes are informed that a card has been moved to the Approved list on Trello.

If a card represents a batch, a set of scripts are applied to first split the AIP into a set of smaller AIPs, one for each item in the batch, then generate individual cards on the Create test DIP list for each item. The batch card is then moved to the Cleanup list.

#### Create DIP shell

The AIP is analyzed. First, the METS file is located. If the package is a finding aid, this file must be named "mets.xml". For historical reasons, the naming convention is much looser for non-collection packages. (This is why there is a validation to verify that the METS file is uniquely identifiable.) For finding aids, the EAD is located using metadata recorded in the METS file.

*Convention*: If there is a finding aid, associated content files will be placed in a directory named after the EAD file (but with the .xml extension removed). The container lists in the leaf elements (furthest from the root) of the container hierarchy are used to generate a list of subdirectories to inspect for possible content files.

*Convention*: If there is not a finding aid, then the first top-level directory (in Linux sort order) appear in the data directory of the bag will be treated as the content directory.

In either case, there is a stream of locations (possibly just one) to look for content files. 

*Convention*: Files in one directory are assumed to be part of a sequence, or manifestations of the same underlying physical object, depending on whether they have the same base name (part of the file name before the extension).

Example. A directory includes the following files:

* 24601.tif
* 24601.txt
* 24601b.tif
* 24602.xml

These are analyzed into the following sequence:

1. 24601, which has two realizations (TIFF and text).
2. 24601b, which has one realization (TIFF).
3. 24602, which has one realization (XML, assumed to be ALTO).

If the package is a finding aid, there will be one Section per subdirectory with content. Otherwise, there will be one Section corresponding to the one content directory.

For each content file, one or more job files are made that specify what transformations (such as converting format or simply copying) should be performed on that file. Here is a sample job file:

```json
{
  "use": "reference image",
  "mime_type": "image/jpeg",
  "target": "/opt/shares/library_dips_2/test_dips/pairtree_root/xt/7g/f1/8s/fq/2v/xt7gf18sfq2v/data/2013ms0843/Box_12/Folder_7d/12_7_1424.jpg",
  "command": "tiff2jpeg",
  "id": "05806beb-22f5-4149-81ef-8b894edfdd20",
  "source": "/opt/shares/library_aips_1/pairtree_root/xt/7m/63/9k/6z/99/xt7m639k6z99/data/2013ms0843/Box_12/Folder_7d/12_7_1424.tif",
  "item_base": "12_7_1424",
  "item": "2013ms0843_Box_12_Folder_7d_12_7_1424"
}
```

The list of possible commands includes:

* tiff2jpeg - convert an image, usually a TIFF, to JPEG. With an additional argument, resize it (as for a thumbnail)
* tiff2pdf - convert a TIFF to PDF
* ocr - extract text from an image using OCR
* pdf2alto - extract bounding box information from a born-digital PDF
* pdf2text - extract raw text from a PDF, if possible
* buildmultipdf - convert a set of images into a multi-page PDF
* copy - simply copy a file from the AIP into the DIP

While these job files are being created, the METS file is also being updated to include a `fileSec` that lists all content files to appear in the DIP and a `structMap` that indicates the organization of the content into Sections and Leaves.

During this process, most of the information that is required to build the MIP—except possibly for OCRed text—is available, and so *ought* to be, but is not, stored at this point. The amount of refined metadata stored in the METS file at this point is quite limited, using the TYPE and LABEL attributes of `<mets:div>` elements to approximate describing Leaves as Audio, Video, Image, or Page. Richer metadata is available at this point, at least for a first automatic approximation, and so we should store that richer metadata in the DIP (or conceptual successor to DIP) at this point.
#### Generate derivatives

After the DIP shell has been created, the next step is to generate the derivatives according to the job files that were provided by the previous step. This is generally the slowest automatic step because of the amount of data that must be processed, and the variety of types of processing that must be applied to that data.

If all derivatives were generated appropriately, the final part of this step is to collect checksums and turn the DIP into a valid BagIt bag. Then the card is moved to Index into test.
### Index into test

For more information on how metadata is extracted, see the [wiki page on ExploreUK metadata, in the Metadata subsection](https://wiki.ukpdp.org/wiki/ExploreUK/Metadata#Metadata) and the [wiki page on the ExploreUK metadata dictionary](https://wiki.ukpdp.org/wiki/ExploreUK/Metadata_dictionary).

The first step of Index into test is to analyze the DIP (without reference to the AIP used for its creation) and extract metadata into JSON files to form the basis of the MIP. This process primarily requires access to the METS and (if present) EAD files, but if there are any text files with transcriptions or OCR available, those will also be used.

Some limited file metadata from other content files is also extracted at this time. In particular, an EXIF reader is used to record width and height for each image.

Each `<mets:div>` in the `structMap` is assigned an identifier which begins with the DIP identifier and includes the ORDER element for the `<mets:div>` and each of its ancestor `<mets:div>` elements, connected with underscores. For example, here is a `<mets:div>` from the [James Edwin Weddle Photographic Collection](https://exploreuk.uky.edu/catalog/xt734t6f3d29), specifically corresponding to the image [Animals; Cats; Close-up of kitten](https://exploreuk.uky.edu/catalog/xt734t6f3d29_115_1).

```xml
<mets:div TYPE="section" LABEL="115" ORDER="115">
	<mets:div TYPE="photograph" LABEL="Animals; Cats; Close-up of kitten" ORDER="1">
		<mets:fptr FILEID="ThumbnailFilee32d581c82b0079697654b79be4347e9"/>
		<mets:fptr FILEID="FrontThumbnailFile104dbe363f0e8f263dab0c52557b64ba"/>
		<mets:fptr FILEID="ReferenceImageFile991ff9a4b7d76b6f8dbee2d7746e58e2"/>
		<mets:fptr FILEID="PrintImageFilef19cc4a2d47746ad4263e294ecaa80ae"/>
	</mets:div>
</mets:div>
```

The DIP identifier is `xt734t6f3d29`, the section's ORDER is `115`, and the photograph's ORDER is `1`. Thus the identifier for this particular image is `xt734t6f3d29_115_1`. Thus all metadata corresponding to this image will be collected into a JSON file named `xt734t6f3d29_115_1`.

If there is a finding aid, then at this point the finding aid will be ingested into the finding aid viewer. (More detail will be added later.)

Finally, the JSON files are converted (using the [Solr document crosswalk](https://wiki.ukpdp.org/wiki/ExploreUK/Metadata#Solr_document_crosswalk)) and indexed into (test) Solr. The package is now completely live in the prepublication or "test" ExploreUK. Once this is done, the card is moved to Ready to review and approving users are tagged on the card o Trello.
## Approval

Trello lists involved:

* Ready for review
* Approved
* More work required

A content submitter will examine the item or items corresponding to a card and determine whether they approve of how the items appear in (test) ExploreUK. If they approve, they signal this by moving the card from Ready for review to Approved. Otherwise, the card should generally be moved to More work required and some indication should be given of what work is required.

Some examples of issues that may block approval:

* some of the images are not showing up, for mystery reasons
* the content within the scope and content note is rendered incorrectly
* there was a typo in the description for this daybook
* there is a directory of additional audio files to add
* most of the descriptions are for the subsequent image
## Publication

Trello lists involved:

* Move DIP to production
* Index into production
* Online
* Cleanup
* Done

Once a card is moved to Approved, its corresponding package is ready to enter the publication process (sometimes called "moving to production").

In the Move DIP to production step, the (test) DIP is copied to the (production) DIP store.

In the Index into production step, the (test) MIP is copied to the (production) MIP store. Then, if there is a finding aid, the finding aid is ingested into the (production) finding aid viewer. Finally, the JSON documents in the MIP are converted and indexed into the (production) Solr index.

Next, the card is moved to the Online list, and approving users are tagged on the card. Sometimes, problems will be identified at this stage!

After a reasonable delay, a repository manager will move a card from Online to Cleanup, and inform the submitter that any submission artifacts can be cleared. (The submission clearing is another potential target for process improvement.)

When a content submitter is done with a package (for now!), they move it to Done. A repository manager regularly archives Done cards. This clears them out of work in progress but keeps them available for further processing or re-processing in the future.