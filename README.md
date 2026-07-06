# DuSub-GS

Dual-Subspace Motion Modeling for Robust Dynamic Gaussian Splatting 


\abstract{
Reconstructing dynamic scenes from monocular videos remains a challenging problem due to motion modeling methods generally suffering from temporal instability, boundary artifacts, or geometric collapse. 
% 
To address this issue, we propose a principled motion decomposition framework for monocular dynamic scene reconstruction based on dynamic Gaussian Splatting. Our method explicitly separates motion into two complementary components: a Chebyshev field that models globally stable, low-frequency motion trends, and a Residual Motion field that captures localized, high-frequency dynamics outside the spectral subspace.
%
The novel Chebyshev field module leverages spectral representations on bounded, non-periodic temporal intervals to ensure temporal coherence and suppress boundary oscillations, while the residual motion field focuses on structured local dynamics without interfering with global stability. We further design a joint optimization objective that enforces appearance consistency, geometric plausibility, and residual regularization, ensuring stable and physically meaningful reconstruction.
%
Extensive experiments on challenging monocular dynamic scenes demonstrate that the proposed framework significantly improves temporal stability, reconstruction fidelity, and motion boundary sharpness compared to existing dynamic Gaussian and neural deformation methods. 
% Our results highlight the importance of explicit motion decomposition as a foundation for robust and interpretable dynamic scene reconstruction.
}
