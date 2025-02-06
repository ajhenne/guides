
# Table of Contents

1.  [Python](#orga00c92e)
    1.  [Rebinning data](#org603e8a4)

Code snippets and useful functions.


<a id="orga00c92e"></a>

# Python


<a id="org603e8a4"></a>

## Rebinning data

    def rebin_data(x, y, x_err, y_err, num_bins):
        # Determine bin edges
        indices = np.linspace(0, len(x), num_bins + 1, dtype=int)
    
        x_binned, y_binned, x_err_binned, y_err_binned = [], [], [], []
    
        for i in range(num_bins):
            start, end = indices[i], indices[i + 1]
    
            x_bin = x[start:end]
            y_bin = y[start:end]
            x_err_bin = x_err[start:end]
            y_err_bin = y_err[start:end]
    
            # Weighted mean using inverse variance weighting
            weights_x = 1 / x_err_bin**2
            weights_y = 1 / y_err_bin**2
    
            x_mean = np.sum(weights_x * x_bin) / np.sum(weights_x)
            y_mean = np.sum(weights_y * y_bin) / np.sum(weights_y)
    
            # Standard error of the weighted mean
            x_err_mean = np.sqrt(1 / np.sum(weights_x))
            y_err_mean = np.sqrt(1 / np.sum(weights_y))
    
            x_binned.append(x_mean)
            y_binned.append(y_mean)
            x_err_binned.append(x_err_mean)
            y_err_binned.append(y_err_mean)
    
        return np.array(x_binned), np.array(y_binned), np.array(x_err_binned), np.array(y_err_binned)

