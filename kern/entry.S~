/* See COPYRIGHT for copyright information. */

#include <inc/mmu.h>
#include <inc/memlayout.h>

# Shift Right Logical 
#define SRL(val, shamt)		(((val) >> (shamt)) & ~(-1 << (32 - (shamt))))


###################################################################
# The kernel (this code) is linked at address ~(KERNBASE + 1 Meg), 
# but the bootloader loads it at address ~1 Meg.
#	
# RELOC(x) maps a symbol x from its link address to its actual
# location in physical memory (its load address).	 
###################################################################

#define	RELOC(x) ((x) - KERNBASE)

###################################################################
# entry point
###################################################################



.text
# Ruogu: Code should start at 0x8000. I have no choice.


# '_start' specifies the ELF entry point.  Since we haven't set up
# virtual memory when the bootloader enters this code, we need the
# bootloader to jump to the *physical* address of the entry point.
.globl		_start
_start = RELOC(entry)

.globl entry
entry:

# Ruogu: Load pgtable to the first one
ldr r0, =RELOC(entry_pgdir)
mcr p15, 0, r0, c2, c0, 0
# Ruogu: use the first table
mov r0, #0
mcr p15, 0, r0, c2, c0, 2
/* Set Domain 0 ACL to "Client", enforcing memory permissions
 * See ARM1176JZF-S manual, 3-64
 * Every mapped section/page is in domain 0
 */
mov r0, #1
mcr p15, #0, r0, c3, c0, #0
/* Read control register to r0*/
mrc p15, #0, r0, c1, c0, #0
/* Turn on MMU */
mov r1, #1
orr r0, r0, r1
/* Enable ARMv6 MMU features (disable sub-page AP) */
mov r1, #(1<<23)
orr r0, r0, r1
/* Write value back to control register */
mcr p15, #0, r0, c1, c0, #0


# Ruogu: Set up boot stack
ldr r0, =bootstack
mov sp, r0

ldr lr, =pi_init
bx lr

.data
###################################################################
# boot stack
###################################################################
# force page alignment
	.p2align	PGSHIFT		
	.globl		bootstack
bootstack:
	.space		KSTKSIZE
	.globl		bootstacktop   
bootstacktop:

