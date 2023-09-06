
# How the patch file was generated

The tag v5.15.130 has some backported(?) changes, from most recent to least recent:

- eaff3946a86fc63280a30158a4ae1e141449817c ath11k: fix netdev open race (upstream commit: d4ba1ff87b17e81686ada8f429300876f55f95ad)
- 8a29aec244ae88a54655d5c4886d7c43b6af401f ath11k: add hw_param for wakeup_mhi (upstream commit: 081e2d6476e30399433b509684d5da4d1844e430)
- 6292bd6f654eb5ac6f45879fdb2af4d080e9c054 ath11k: add string type to search board data in board-2.bin for WCN6855 (upstream commit: fc95d10ac41d75c14a81afcc8722333d8b2cf80f)

Excluding the above changes, the history of the tag appeared linear up to commit 515bda1d1e51c64edf2a384a58801f85a80a3f2d: ath11k: Fix an error handling path in ath11k_core_fetch_board_data_api_n(). On top of this commit, we need to add up to and including the commit that adds the support for the card we are using d1147a316b53df9cb0152e415ec41dcb6ea62c1c:  ath11k: add support for WCN6855 hw2.1. So, we revert the backports, and then linearly cherry pick our way up to this commit. Then we add the backports on top again.

```bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
cd linux
git checkout v5.15.130
# Revert from top of the stack
git revert eaff3946a86fc63280a30158a4ae1e141449817c
git revert 8a29aec244ae88a54655d5c4886d7c43b6af401f
git revert 6292bd6f654eb5ac6f45879fdb2af4d080e9c054
# Now, we should be at the same code base as 515bda1d1e51c64edf2a384a58801f85a80a3f2d
# Add the commits on top, till d1147a316b53df9cb0152e415ec41dcb6ea62c1c
git cherry-pick cc2ad7541486f1f755949c1ccd17e14a15bf1f4e
git cherry-pick 1cae9c0009d35cec94ad8e1b06ebcb2d704626bf
git cherry-pick b72e86c07e9881d249fbb7511060692f3fb6b687
git cherry-pick c72aa32d6d1c04fa83d4c0e6849e4e60d9d39ae4
git cherry-pick 31582373a4a8e888b29ed759d28330a1995f2162
git cherry-pick 734223d78428de3c7c7d7bc04daf258085780d90
# For some reason this had a conflict. Simply accept their change
git cherry-pick --strategy=recursive -X theirs 82c434c103408842a87404e873992b7698b6df2b
git cherry-pick b2beffa7d9a67b59b085616a27f1d10b1e80784f
git cherry-pick 6452f0a3d5651bb7edfd9c709e78973aaa4d3bfc
git cherry-pick f951380a6022440335f668f85296096ba13071ba
# The following two are the upstream commit hashes for reverted backports
git cherry-pick fc95d10ac41d75c14a81afcc8722333d8b2cf80f
git cherry-pick 081e2d6476e30399433b509684d5da4d1844e430
# There are two commits here that I skipped because the second reverts the first
# THE main commit we need
git cherry-pick d1147a316b53df9cb0152e415ec41dcb6ea62c1c
# We have the commit that we need; now restore the pending backport
git cherry-pick d4ba1ff87b17e81686ada8f429300876f55f95ad
# Stop AMD log spam
git revert 187916e6ed9d0c3b3abc27429f7a5f8c936bd1f0
git diff v5.15.130 > 0204_ath11k_pci-wcn6855_amdgpu_vm.patch
```
