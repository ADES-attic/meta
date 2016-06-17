# Roadmap to initial deployment

This describes some anticipated work to be completed before an initial deployment of the ADES.

## Deployment

First, what would "deployment" mean?  Perhaps that the MPC begins accepting ADES observations, that at least one observer beins routinely submitting ADES observations, and that the MPC is processing the ADES observations.

Someone suggested that it might also mean that the MPC is distribuiting ADES observations.  That would be nice, but it could conceivably come as a later stage.

## Required work

### Changes to the ADES
Changes are being considered.  Any changes made before initial deployment will trigger an update cycle to update all existing code and documentation, for example the schemas in the xsd repo, the utilities and libraries in the C repo, and internal software at the MPC and at submitting sites.

### Schema fixes
Consider issues filed in the xsd repo.  At initial deployment the MPC will validate submissions against the submit schema so that at least needs to have issues resolved.  As the submit schema is likely to be based (somehow) on the exchange schema, that too should have issues resolved.

### "Lists provided by the MPC"
* Describe the format and location of these lists.  (consider catalog support)
* The MPC must then actually publish these lists as described

## Pretty important work

## Testing
Develop data sets representing all variations described by the ADES, run them through schema validation and stand-alone utilities.

## Catalog support
This is an XML thing potentially useful for the lists provided by the MPC.  It describes caching lists at a user site where documents are validated so the lists can be used in validation.

## Other work that would be nice

## Merge utility
Listed in the "Software recommendations" document (SR) but not yet written.  This is relevant the use-case of combining multiple ADES data files for purposes like orbit computation.

## Sort utility
Also in SR, and also applicable to the orbit computation use-case.  It could be combined with the merge utility but the unix way of "do one thing well" is probably better.

## Extract utility
SR mentions splitting files.  An xpath based extractor might be one way to do this.

## Platform support
Utilities are currently C source tarballs, developed on Linux and tested on a couple of Linux systems.  For other platforms, testing or development of more platform-friendly ways of distribution would require someone with physical access to the platforms, likely some purchased development kits for the platforms, and ideally some knowledge, skills, and experience on the platforms.

## Streaming utilities
Existing utilities load entire files into memory.  This could be problematic for use-cases involving archival data sets for example.  Utilities might be upgraded to support streaming or variant utilities could be produced to support streaming.
