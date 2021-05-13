# IIPs
ICON Improvement Proposals (IIPs) describe standards for the ICON platform, including core protocol specifications, client APIs, and contract standards.

# Contributing

 1. Review [IIP-1](IIPS/iip-1.md).
 2. Fork the repository by clicking "Fork" in the top right.
 3. Add your IIP to your fork of the repository. There is a [template IIP here](iip-X.md).
 4. Submit a Pull Request to ICON's [IIPs repository](https://github.com/icon-project/IIPs).

Your first PR should be a first draft of the final IIP. It must meet the formatting criteria. An editor will manually review the first PR for a new IIP and assign it a number before merging it. Make sure you include a `discussions-to` header with the URL to a discussion forum or open GitHub issue where people can discuss the IIP as a whole.

If your IIP requires images, the image files should be included in a subdirectory of the `assets` folder for that IIP as follow: `assets/iip-X` (for iip **X**). When linking to an image in the IIP, use relative links such as `../assets/iip-X/image.png`.

Once the first draft of IIP is merged, only the PRs from the owner of the draft will be accepted. Make sure that the 'author' line of your IIP contains either your Github username or your email address inside <triangular brackets>. If you use your email address, that address must be the one publicly shown on [your GitHub profile](https://github.com/settings/profile).

When you believe your IIP is mature and ready to progress past the draft phase, you should do one of two things:

 - **For a Standards Track IIP of type Core**, ask to have your issue added to [the agenda of an upcoming All Core Devs meeting](https://github.com/icon-project/pm/issues), where it can be discussed for inclusion in a future release. If implementers agree to include it, the IIP editors will update the state of your IIP to 'Accepted'.
 - **For all other IIPs**, open a PR changing the state of your IIP to 'Final'. An editor will review your draft and ask if anyone objects to its being finalised. If the editor decides there is no rough consensus - for instance, because contributors point out significant issues with the IIP - they may close the PR and request that you fix the issues in the draft before trying again.

# IIP Status Terms
* **Draft** - an IIP that is open for consideration.
* **Last Call** - an IIP that is calling for last review before finalizing. IIPs that has been more than 2 weeks in Last Call without any technical changes or objections enters either Accepted or Final state.
* **Accepted** - an IIP that is planned for immediate adoption, i.e. expected to be included in the next release (for Core/Consensus layer IIPs only).
* **Final** - an IIP that has been adopted. For Core/Consensus layer IIPs, the implementation has been adopted in the mainnet.
* **Deferred** - an IIP that is not being considered for immediate adoption. May be reconsidered in the future.

# IIPs

| Number             | Title                      | Author    | Type | Status |
| ------------------ | -------------------------- | --------- | ---- | ------ |
| [1](IIPS/iip-1.md) | IIP Purpose and Guidelines | Sojin Kim | Meta | Active |
| [2](IIPS/iip-2.md) | ICON Token Standard | Jaechang Namgoong  | IRC | Final |
| [3](IIPS/iip-3.md) | ICON Non-Fungible Token Standard | Jaechang Namgoong  | IRC | Final |
| [6](IIPS/iip-6.md) | ICON Name Service Standard | Phyrex Tsai, Portal Network Team | IRC | Draft |
| [14](IIPS/iip-14.md) | ICONex Connect for Mobile | Jeonghwan Ahn | IRC | Final |
| [16](IIPS/iip-16.md) | ICON Security Token Standard | Patrick Park, Justin Hsiao | IRC | Draft |
| [25](IIPS/iip-25.md) | ICON BTP Standard | MoonKyu Song | IRC | Draft |
| [31](IIPS/iip-31.md) | ICON Multi Token Standard | Jaechang Namgoong | IRC | Draft |
| [35](IIPS/iip-35.md) | ICON BTP Fee Gathering | Heonseung Lee | IRC | Draft |
