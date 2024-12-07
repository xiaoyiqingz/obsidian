---
title: cobra cli
draft: true
tags:
  - ptc
---

```go
	mainCmd = &cobra.Command{
		Version: Version,
		Use:     "ptc",
		Short:   "ptc",
	}

	for cmd := cmds {
		parentCmd = &cobra.Command{
			Use:   cmd.Use(),
			Short: cmd.Short(),
			Run: func(cmd *cobra.Command, args []string) {}
			PreRunE: func(cmd *cobra.Command, args []string) error {}
			PostRunE: func(cmd *cobra.Command, args []string) error {}
		}
		
		if cmd.Extend().Subcommands != nil {
			for subCmd := cmd.Extend().Subcommands {
				childCmd = &cobra.Command{ ... }

				parentCmd.AddCommand(parentCmd)
			}
		} 
	
		mainCmd.AddCommand(parentCmd)
	}
```
如上 共有三级
1.  mainCmd, 整个cmd结构 如 `ptc, ptc --help`
3. 第一级cmd 对应命令 ptc test  如 `ptc make`
4. 第二级cmd 对应具体命令 ptc test subTest  如 `ptc make 01`